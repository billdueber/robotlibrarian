---
title: "Requiring/Preferring searches that don't span multiple values (SST #3)"
date: 2012-03-09
tags: "phrase slop, solr, Stupid Solr Tricks"
layout: post
slug: requiringpreferring-searches-that-dont-span-multiple-values-sst-3

---

> Check out [introduction to the Stupid Solr Tricks series](http://robotlibrarian.billdueber.com/stupid-solr-tricks-introduction/) if you're just joining us.]

### Solr and multiValued fields

Here's another thing you need to understand about Solr: it doesn't really have fields that can take multiple values.

"But Bill," you're saying, "sure it does. I mean, hell, it even has a 'multiValued' parameter."

First off: watch your language.

Second off: are you *sure*?

Let's do a quick test. Look at the following documents


~~~javascript
exampledocs/names.json
[
  {
    "id": "1",
    "title": "The Monkees",
    "name_text": ["Peter Tork", "Mike Nesmith",
                  "Micky Dolenz", "Davy Thomas Jones"]
  },
  {
    "id": "2",
    "title": "Heros of the Wild West",
    "name_text": ["Buck Jones", "Davy Crockett"]
  }
]

~~~

Question: what do you get when you run this query against those two documents?


~~~ruby
ruby/names_query.rb
{
  'fl' => 'score, *',
  'defType' => 'dismax',
  'wt' => 'csv',
  'qf' => 'name_text',
  'q' => 'davy jones'   # Poor guy just died. So young. So short.
}

~~~

See how I threw the _wt=csv_ in there? Check out [all the query response formats](http://lucene.apache.org/solr/api/org/apache/solr/response/QueryResponseWriter.html) if you're interested, but really all you'll use is `standard` (XML), `json`, or `csv` unless you're rolling your own in some way.

I've updated `ruby/browse.rb` to allow a second argument of the type of output you want. You can now do `ruby browse.rb jsonfile [json|csv|standard|xml]`


### Following along at home?

If so, let's go ahead and index these document and run the query.


~~~bash
Play along at home
cd solr_stupid_tricks
git pull origin master
git fetch --all
git checkout SST3 # I've started tagging the repo for these posts
# ignore warning about "detached HEAD"
java -jar start.jar &
cd exampledocs
 ./reset_and_index_json.sh names.json
 cd ../ruby
 ruby browse.rb names_query.rb

~~~

Here's the scores that I get:

~~~csv
Return from Solr
  id,title,name_text,score
  2,Heros of the Wild West,"Buck Jones,Davy Crockett",0.42039964
  1,The Monkees,"Peter Tork,Mike Nesmith,Micky Dolenz,Davy Thomas Jones",0.26274976

~~~

Check out that last column. The query was _davy jones_. Document #1 contains a name that has both those terms, but document #2 (which has both terms, but in different names) gets a higher score.

### The relevance ranking seems...wrong

While it _looks_ like we added four separate names to the `name_text` field in our first document, Solr doesn't see it that way. Solr treats those four poor Monkees as if they had one long name.

Then it finds all the documents that match the query (both of our documents match) and figures out which is a better match by assigning a score.

In this case, while both document have both query terms, the field in the second document is _shorter_. Which means that, essentially, a _higher percentage of the terms in the field value match the given  query terms_. In Solr's mind, that makes it a better match, and the shorter document shows up first.

Solr doesn't automatically give more weight to the recently-dead Monkee because internally it doesn't care that you're thinking of those values as four separate names. It just concatenates them together and indexes them.

This is **not**, for most people, expected behavior.

### Phrase slop

Part of what's going on here is that we haven't told Solr that it should care how close together the terms are.

One way to do that is to use a phrase query by throwing quotes around the terms


~~~ruby
Put double-quotes around it to make it a phrase query
  "q" => '"Davy Jones"'

~~~

...but that won't find anything, because _Davy_ and _Jones_ aren't right next to each other in our document.

Solr does allow a phrase query to be "sloppy", though -- basically saying that instead of being right next to each other, the terms need to be within a certain number of tokens of each other.

For that, we'll tell solr to search against certain fields (`pf`) treating the query as a phrase, and allow a little slop (`ps`) as well.


~~~ruby
ruby/names_sloppy_query.rb
  {
    'fl' => 'score, *',
    'defType' => 'dismax',
    'wt' => 'csv',
    'q' => 'davy jones',
    'qf' => 'name_text',
    'pf' => 'name_text^10', # search this field as a phrase
    'ps' => '4' # allow 'phrase' to mean 'within 4 tokens of each other'
  }  

~~~

That gets us something more expected.


~~~csv

  id,title,name_text,score
  1,The Monkees,"Peter Tork,Mike Nesmith,Micky Dolenz,Davy Thomas Jones",0.2806283
  2,Heros of the Wild West,"Buck Jones,Davy Crockett",0.029652705

~~~

### Enter `positionIncrementGap`

OK. Now that we have the concept of "slop", one of those mystery `fieldtype` parameters makes sense: `positionIncrementGap`. Basically, a `positionIncrementGap` of 1000 means _When computing slop, pretend there are 1000 tokens between the entries in a multValued field_.

A sloppy phrase search, then, will only find (and thus boost) the phrase if (a) the tokens are in the same entry for a multiValued field, and (b) your slop value is less than your `positionIncrementGap`.

All you have to do is use the `pf` and `ps` parameters and you're set.

Note that this should be telling you two things:

* **Always use the same positionIncrementGap for your multiValued fields**
* **Make it a number much larger than the maximum number of tokens you expect to ever have in a field.**

Note that a large `positionIncrementGap` doesn't _actually_ put 1000 tokens in there -- a large value doesn't affect processing time or your index size or anything.

### But I'm already using the `pf` parameter!

Slop is great when you want it. But I don't always want to use slop. Slop of 4 makes the phrase "_Sex in the City_" be treated exactly the same as "_In the Sex City_". If someone puts in an exact title, I want to reward them for that query by floating the exact match to the top, and slop prevents me from doing so.

[_Forshadowing_: We'll talk about exact-ish matches in a few days.]

OK, so we can't just appropriate the `pf`/`ps` parameters and and push the slop value up all the time -- that cripples our ability to create the query boost structure we want.

### Query slop

So, dismax (and its cousin edismax) have an analogous parameter that affects only _phrases within_ the normal query: `qs`.

`qs` is a dismax param that affects _query slop_ -- how much slop to allow in phrases within the query, much like the `ps` param.


The query


~~~ruby
A three-token query
  'q' => 'Bill "The Weasel" Dueber'

~~~

...has three tokens, the second of which (_"The Weasel"_) is a phrase. It's that phrase token that is affected by query slop.

OK. So it affects only the phrases in the normal query. But...suppose we just force the *entire* query to be one big phrase? That'll get us somewhere!

We just need to do the following:

* Create a boost query that uses the same fields as the regular query
* ...but treats all the query terms as one big phrase
* ...and give it a query slop of one less that the `positionIncrementGap` in our field type definition (in my case, 999)


### Package it up

OK, so here's what we're going to do. You can just take this basic idea and build it into your own queries in your application code. Try it. You might like it. Play around with what fields are affected, how much weight to give it, etc.

But heck, we've gone this far. Let's encode it into the Solr configuration file `solrconfig.xml` itself as a custom request handler.

We're going to extend our `edismaxplus` requestHandler from [last time](http://robotlibrarian.billdueber.com/using-localparams-in-solr-sst-2/), but we'll add an extra boost query that reflects this new "prefer documents where all the tokens appear in the same 'line' of a multiValued query" attitude.


**solr/conf/solrconfig.xml**

~~~xml
<requestHandler name="/edismaxplus" class="solr.SearchHandler">
  <lst name="defaults">
    <str name="rows">10</str>
    <str name="fl">*,score</str>
    <str name="echoParams">explicit</str>
    <str name="q">
      _query_:"{!edismax qf=$fields mm=$mymm
                          v=$qwords bq=$boostForAll}"</str>
    <str name='mymm'>0%</str>
    <str name="qwordsphrase">"JunkThatWillNEverShowUpInAMillionFreakinYears"</str>
    <str name='boostForAll'>
      _query_:"{!edismax qf=$fields
                         mm='100%'
                         v=$qwords }"^5 OR
      _query_:"{!dismax  qf=$fields
                         mm='100%'
                         v=$qwordsphrase
                         qs='999'}"^5
    </str>
  </lst>
</requestHandler>  

~~~

We now do a few new things:

* (_Line 15_) Add a second clause to the boost query that use the same fields provided for the regular query (note the boolean OR between the two localparams queries that comprise this boost query)
* (_Line 17_) Ask for another user-provided value: `qwordsphrase` which your application-level stuff should set to the set of all the regular query ters, but as a single phrase. Basically, strip out all the double-quotes, then put the whole thing in double quotes. In ruby: `qwordsphrase = '"' + qwords.gsub(/"/, '"') + '"'`
* (_Line 10_) Provide a default value for the new `qwordsphrase` that won't ever show up in a real query (empty string won't work; I tried it and it throws an error). So, if the application doesn't provide `qwordsphrase`, no harm is done -- the search regresses to what we had last time.
* (_Line 18_) Use a `qs` (query slop) of 999 in the new boost clause acting against `qwordsphrase`. That value is one less than the `positionIncrementGap` of 1000, making sure that we don't cross multiValue boundaries.

**Note**: If you wanted to, you could make this a filter query (`fq`) instead of a boost query to _only_ allow documents that meet this criterion.

### Let's try it out!

Once again, if you did a `git pull origin master` you've got this up and running already -- the updated requestHandler source is already in `solr/conf/solrconfig.xml`.

We first construct the query just like we did last week, without the `qwordsphrase` argument:

 [http://localhost:8983/solr/edismaxplus/?qwords=davy jones&amp;fields=name_text](http://localhost:8983/solr/edismaxplus/?qwords=davy%20jones&amp;fields=name_text)

You'll see Davy Crockett and friend appear as the first item.

But when you add the phraseified query, you'll see the boost we've been talking about this whole post and get something more expected.

 [http://localhost:8983/solr/edismaxplus/?qwords=davy jones&amp;fields=name_text&amp;qwordsphrase="Davy Jones"](http://localhost:8983/solr/edismaxplus/?qwords=davy%20jones&amp;fields=name_text&amp;qwordsphrase=%22Davy%20Jones%22)

The Monkees are again on top! Party like it's 1967!

### Where it breaks down

If you actually have a phrase as one of your query terms, it will no longer be treated as a phrase during the boost because we're getting rid of all the double-quotes.

And, of course, if you've got gobs of full-text and include your fulltext field, setting  query slop to 999 isn't just a cute trick, it's a cute trick that will melt your servers to slag and still not do what you want it to do.

### What have we learned?

* Solr doesn't really separate multiple values from each other in a `multiValued` field
* Phrase slop (`ps`) and query slop (`qs`) can be used to allow "phrase" to mean "a bunch of tokens within X spots of each other"
* _I'm A Believer_ is the best song Neil Diamond ever wrote.
