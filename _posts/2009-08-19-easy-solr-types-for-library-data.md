---
title: "Easy Solr types for library data"
date: 2009-08-19
---

[Yet another bit in a series about our Vufind installation]

While I'm no longer shocked at the terrible state of our data *every single day*, I'm still shocked pretty often. We figured out pretty quickly that anything we could do to normalize data as it went into the Solr index (and, in fact, as queries were produced) would be a huge win.

There's a continuum of attitudes about how much "business logic" belongs in the database layer of any application. Some folks -- including super-high throughput sites, but mostly people who have never used anything by MySQL -- tend to put *no* logic into the database. I've always edged over the middle to the other side of that debate, preferring to let the database do type-checking and conversions and track foreign keys and the like. 

Solr, while not a traditional RDBMS, offers this type of functionality in its text filters. One can pipe data through a few standard filters, or write a custom one in Java if need be. The nice part is that it applies at index *and query* time. One obvious application, which I somehow haven't bothered to write yet, is to convert all ISBNs to 13-characters new-style ISBNs upon both index and query. That way, you don't care if your original records had the short or long form; all the data gets converted no matter how it comes in.

Our standard text field is similar to the default schema.xml, for example, running text through the following filters:

* UnicodeNormalization to normalize unicode composition and (optionally) remove diacritics
* StopFilter to ignore stopwords in a separate file
* WordDelimiter to do intelligent word deliniation
* LowerCase to...you know...lowercase everything
* EnglishPorter to do stemming
* RemoveDuplicates to do what it says

And because it happens on index and on query, everything works out. 

We're running Solr basically from trunk -- whenever we need to change something, I pull down a fresh svn copy, put in our local changes to make sure it all works, and then deploy -- so I have access to stuff slated for Solr 1.4, including most importantly Trie fields and the  PatternReplaceFilterFactory.

The stdnum type
-------------------------

One of the first things we defined was a "stdnum" type, to deal with supposedly-unique identifiers, possibly with embedded dashes and dots and leading/trailing nonsense. Here's a variant. 


~~~xml

  <fieldType name="stdnum" class="solr.TextField" sortMissingLast="true" omitNorms="true" >
    <analyzer>
      <tokenizer class="solr.KeywordTokenizerFactory"/> 
      <filter class="solr.LowerCaseFilterFactory"/>
      <filter class="solr.TrimFilterFactory"/>
      <filter class="solr.PatternReplaceFilterFactory"
           pattern="^[\D]*([\d\-\.]+x?).*$" replacement="$1"
      />
      <filter class="solr.PatternReplaceFilterFactory"
           pattern="[^\dx]" replacement=""  replace="all"
      />
      <filter class="solr.PatternReplaceFilterFactory"
           pattern="^0+" replacement=""  replace="all"
      />
    </analyzer>
  </fieldType> 

~~~

Let's walk through it. It could probably be done in one go, but solr is not our bottleneck at this point...

* We start by defining it as a *TextField* because it's the only type that can take filters. 
* We then declare that instead of the standard tokenizer, we're using the *KeywordTokenizer*. Confusingly, the KeywordTokenizer doesn't tokenize in the traditional sense -- it just returns the whole input _as a single token_. 
* Lowercase it.
* Trim spaces off both ends
* Skip any leading non-digts, find a string of numbers, dashes, and dots, with optional x at the end, and skip everything after it.
* Remove anything left that isn't a digit or an 'x'.
* Remove leading zeros, if you've got 'em. 


The net effect is a trimmed string that has only digits (with an optional trailing 'x') and removes any leading zeros. 

We use this "stdnum" field for ISBNs and ISSNs (and I think OCLC numbers) and it should work for any messy numerics you might have lying around. If you wanted to, you could change the regexp to enforce a minimum string of digits so it doesn't get confused by any leading nonsense, e.g, "ISSN2: 1234567X (online)". But if your data are that bad, you may have bigger problems to worry about. 

textProper type
---------------
We define a _textProper_ type that is exactly the same as the default text type, but without the stemming and synonyms. In the presence of stemming, exact matches and stemmed matches count the same toward relevancy (e.g. *row* and *rowing*). We had plenty of examples where exact results were getting overridden by the stemmed results, and this is confusing.

So for most of our important fields, we index them as both *text* and *textProper* so we can apply different weights to searches against them.

By the way, don't forget to make sure your authors are in a textProper type; you don't want stemming on author names!

exactmatcher type
------------------
The name *exactmatcher* is a red herring, of course, It's not an exact matcher. It just strips out all the delimiters so we can pretend it's an exact match.


~~~xml

  <fieldType name="exactmatcher" class="solr.TextField" omitNorms="true">
    <analyzer>
      <tokenizer class="solr.KeywordTokenizerFactory"/> 
      <filter class="schema.UnicodeNormalizationFilterFactory" version="icu4j" composed="false" remove_diacritics="true" remove_modifiers="true" fold="true"/>
      <filter class="solr.LowerCaseFilterFactory"/>
      <filter class="solr.TrimFilterFactory"/>
      <filter class="solr.PatternReplaceFilterFactory"
           pattern="[^\p{L}\p{N}]" replacement=""  replace="all"
      />
    </analyzer>
  </fieldType>

~~~

That's it. Lowercase it, normalize the unicode, and pull out everything that's not a (unicode) letter or number. 

Note that we're still using *KeywordTokenizerFactory* -- we're getting exactly *one* token out of this thing. That means that the query input either matches (as one string) or it doesn't. 

Here's how we use it:

* *Control numbers*: Our controlnums (old ids, that sort of thing), report numbers, sdr numbers (related to HathiTrust), the HathiTrust ID
* *Callnumbers*: I also try to normalize LC, but this helps people find everything else
* *Titles*: in addition to a regular tokenized title, we index the 245a (as *title\_a*) and the 245ab (as *title\_ab*). If someone types in an exact match for either of them, we shoot the relevancy through the roof (more so for the title\_ab than the title\_a, obviously). This makes known item searching a little less painful.

wildcard searching
------------------
One downside of using all these filters is that **Solr ignores filters when doing wildcard searches**. There is a patch floating around that will using an analyzing query parser for wildcard searches, but I haven't had time to fiddle around with it.

One thing you can do is to do the *exact same normalization* in your calling code and then throw a '\*' on the end of it. The data are in the index, after all -- you just have to do the filtering yourself. For example, for a cheap and easy "Title starts with" search, you can do the same normalization in PHP or Ruby or whatever as we do in the Solr exactmatcher type, drop a '\*' on the end of it, and query against the _exactmatcher_ version of your title. Voila. 

Custom filters
--------------
Regular expressions can get you ridiculously far, but for a couple cases it'd be nice to have custom code running. I've already mentioned that we should by upcasting all our ISBNs to the 13-character variant. The other two areas where I do this are to [normalize LCCNs](http://www.loc.gov/marc/lccn-namespace.html#syntax) and to badly normalize LC CallNumbers. I'll talk about both soon.
