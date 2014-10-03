---
title: "Boosting on Exactish (anchored) phrase matching in Solr: (SST #4)"
date: 2012-03-19
tags: "exact, fieldtype, solr, Stupid Solr Tricks"
---

> Check out [introduction to the Stupid Solr Tricks series](http://robotlibrarian.billdueber.com/stupid-solr-tricks-introduction/) if you're just joining us.]

_Exact_ matching in Solr is easy. Use the default _string_ type: all it does is, essentially, exact phrase matching. _string_ is a great type for faceted values, where the only way we expect to search the index is via text pulled from the index itself. Query the index to get a value: use that value to re-query the index. Simple and self-contained.

But much of the time, we don't want exact matching. We want _exactish_ matching. You know, where things are exactly the same _except_. Except for case, or punctuation, or how much whitespace is between tokens. Maybe do some unicode folding, or stemming. 

Essentially, we want to reward users (via high relevancy) for getting _really close_. If someone types in a full title, but misses a colon, well, let's go  ahead and assume they want that particular item.

### _Exactish_ matching vs phrase matching

Phrase matching in Solr does a great job, but fails those of us generating super-complex queries where we want to provide awesome service for those users doing _known-item queries_. If someone puts in the exact(ish) title, or the exact(ish) subject, well, those items should float to the top.

Solr's default phrase matching (via, say, the `pf` param in dismax or just putting your query in quotes) doesn't differentiate between a phrase that matches the whole target string and only part of that target string. For this, we'll need a decent text `fieldtype` and a way to "anchor" the search to both ends of the target string.

### Our goals

We're shooting for:

* A useful text type that we can use all over the place
* A phrase match against that field that will match any portion of the target text. Solr already does this -- that's a normal Solr phrase search.
* A "fully anchored" text type that will only phrase match if the query string exactishly-matches the whole field. We'll phrase-search on this field and boost it way up.
* And, what the heck, a left-anchored version that will exactish match a phrase only at the start of a field. We'll boost this one up a bit less.


### Follow along at home

Go ahead and clone the [github repo](https://billdueber@github.com/billdueber/solr_stupid_tricks) I've been using  if you haven't already and let's dig in.


~~~bash

cd solr_stupid_tricks
git pull origin master
git fetch --all
git checkout SST4
java -jar start.jar &

~~~

There are some additions to the `schema.xml` file; let's take a look!

### Step 1: get a decent text type

The recent-nighty of Solr 3.x we're using has a great tokenizer in ICUTokenizerFactory, which does "the right thing" across a whole host of languages. 


~~~xml

<fieldtype name="text" class="solr.TextField" positionIncrementGap="1000">
  <analyzer>
    <tokenizer class="solr.ICUTokenizerFactory"/>
      <filter class="solr.ICUFoldingFilterFactory"/>
      <filter class="solr.SynonymFilterFactory" 
              synonyms="syn.txt" ignoreCase="true" expand="false"/>
<!-- <filter class="solr.WordDelimiterFilterFactory" 
             generateWordParts="1" generateNumberParts="1" 
             catenateWords="1" catenateNumbers="1" catenateAll="0"/> -->
      <filter class="solr.CJKWidthFilterFactory"/>
      <filter class="solr.CJKBigramFilterFactory"/> 
  </analyzer>
</fieldtype>

~~~

Let's take it bit by bit:

* Obviously, start with the `ICUTokenizer` with a large positionIncrementGap so we can do some of the tricks we talked about [last time](http://robotlibrarian.billdueber.com/requiringpreferring-searches-that-dont-span-multiple-values-sst-3/)
* Next, we get one-stop shopping with the `ICUFoldingFilterFactory`. It provides all of the following:
  * NFKC normalization (precomosing), 
  * Unicode case folding (i.e., lowercasing)
  * search term folding (removing accents, etc).
* Push in synonyms if you have any
* Uncomment the `WordDelimiterFilterFactory` if you want to. I'm going to try to avoid it, since it messes with the number of tokens midstream and I worry about the effect on dismax and its `mm` parameter as [explained so excellently by Jonathan Rochkind](http://bibwild.wordpress.com/2011/06/15/more-dismax-gotchas-varying-field-analysis-and-mm/)
* [Dealing with CJK (Chinese, Japanese, Korean) is hard](http://www.hathitrust.org/blogs/large-scale-search/multilingual-issues-part-1-word-segmentation). The CJK filters process those languages and provide overlapping bigrams so searching isn't (I'm told) quite as painful. (I really, really recommend the above link for a great overview by Tom Burton-West).


### Step 2: Set up parallel text types that anchor phrase matches to one or both ends

We're going to use something new: a `charFilter`. This differs from a normal filter in that it affects the input string before tokenization. 

Here's the trick. We're going to add anchoring text (I chose just 'AAAA' at the front and 'ZZZZ' at the end) to the normal text type, just by adding a simple charfilter.


~~~xml

<fieldtype name="text_lr" class="solr.TextField" positionIncrementGap="1000">
  <analyzer>
    <charFilter class="solr.PatternReplaceCharFilterFactory"
      pattern="^(.*)$" replacement="AAAA $1 ZZZZ" />       
    <tokenizer class="solr.ICUTokenizerFactory"/>
      <filter class="solr.ICUFoldingFilterFactory"/>
      <filter class="solr.SynonymFilterFactory" 
              synonyms="syn.txt" 
              ignoreCase="true" expand="false"/>
      <filter class="solr.CJKWidthFilterFactory"/>
      <filter class="solr.CJKBigramFilterFactory"/> 
  </analyzer>
</fieldtype>
  

~~~

Note that this charFilter actually adds two new tokens ('AAAA' and 'ZZZZ') to your token stream on both index and query. How does this help us?

Let's look at indexing `Mister Blue Sky` in a normal text field. A normal solr phrase query `q="Blue Sky"` will match on that value, because the query phrase is fully contained in the indexed phrase. 

But what happens if we index into a `text_lr` field?

* Indexing `Mister Blue Sky` becomes `aaaa mister blue sky zzzz`
* Search terms `blue sky` becomes `aaaa blue sky zzzz`
* Phrase searching will then compare the two transformed values using normal Solr rules, _find the the latter is not fully contained in the former as a phrase_, and give up.

Be careful, though. That 'aaaa' and 'zzzz' are there just as if you'd typed them in. Thus every indexed value has the tokens 'aaaa' and 'zzzz', and every query will, in effect, include a query for 'aaaa' or 'zzzz' (depending on your `mm` settings). 

That means that **any non-phrase query will match every field that uses this fieldtype**, and it will also mess with token counts with respect to your `mm` parameter. For those reasons, _only ever use anchored fieldtypes for phrase queries when you want exactish matches_. 

By adding only one of 'AAAA' or 'ZZZZ', we can have left-anchored and right-anchored searches as well. See [the schema.xml](https://github.com/billdueber/solr_stupid_tricks/blob/SST4/solr/conf/schema.xml) for these definitions.

### Try it out!

Let's take a small set of new documents:


~~~ruby

[
  {
    "id": "1",
    "title": "The Monkees: Pleasant Valley Never"
  },
  {
    "id": "2",
    "title": "The Monkees"
  },
  {
    "id": "3",
    "title": "Meet the Monkees"
  },
  {
    "id": "4",
    "title": "Corportate boy bands through the ages"
  }
]

~~~

We have copyFields set up to copy the title field to both a fully-anchored field (`text_exact`) and a left-anchored field (`text_l`).


~~~xml

  <copyField source="title" dest="title_exact"/>
  <copyField source="title" dest="title_l"/>

~~~

If you're following at home, clear out your solr and index them:


~~~bash

cd exampledocs
 ./reset_and_index_json.sh exactish.json 

~~~

We'll now run three dismax queries, all of which use the search terms `the monkees`. Watch what happens to the score as we change things.

* First, `qf=title, pf=title^2`. This will match the three Monkees documents, and then boost *all* of them because they all contain the phrase "the monkees" in the title.
* Second, `qf=title, pf=title_exact^10 title^2`. These will match the Monkees documents, and then give a huge boost to the one with the exact match.
* Finally, `qf=title, pf=title_exact^10 title_l^5 title^2`. There you'll see the score for the exact title match go way up (relatively speaking, of course), and document 1 go up quite a bit (because it begins with the phrase "The Monkees").

You can run all three queries as:


~~~bash

cd ruby
ruby browse.rb exactish_query.rb 
# or ruby browse.rb exactish_query.rb json|xml|csv to get different output type

~~~

[BTW, `browse.rb` will now take an array of queries to run in a single file.]


Tah Dah! You've successfully boosted the exatish match, and the left-anchored exactish match. Your known-item-searchers will thank you.

You may want to take a look at `exactish_query.rb` to see what's going on.

### To sum up

* Your `schema.xml` now contains a decent text type and three variants for anchoring phrase searches left, right, and full (exactish)
* The anchored text fields should NOT NOT NOT be searched against by anything other than a single phrase (which means they're very useful in the `pf` param of a dismax search). A non-phrase search will trivially match *every single document*, so, you know, avoid that.
* You now have a set of tools (field types, copyField directives, phrase search) that can be used to provide higher boosts to exactish matches and left-anchored exactish phrase matches.
