---
title: "Solr: Forcing items with all query terms to the top of a Solr search"
date: 2010-08-18
---

[Note: I've since made a [better explanation of, and solution for, this problem](http://robotlibrarian.billdueber.com/using-localparams-in-solr-sst-2/).]

Here at [UMich](http://lib.umich.edu/), we're apparently in the minority in that we have [Mirlyn](http://mirlyn.lib.umich.edu/), our catalog discovery interface (a very hacked version of [VuFind](http://vufind.org/)), set up to find records that match only a subset of the query terms.

Put more succinctly: everyone else seem to join all terms with 'AND', whereas we do a [DisMax](http://www.lucidimagination.com/blog/2010/05/23/whats-a-dismax/) variant on 'OR'.

Now, I'm actually quite proud of how our searching behaves. Reference desk anecdotes and our statistics all point to the idea that people tend to find what they're looking for. I invite you to try [our current configuration](http://mirlyn.lib.umich.edu/) out -- and, of course, let me know if something feels off to you. We have control of our OPAC now, and can actually fix things.

### The "problem": DisMax is weird

The DisMax algorithm is complex. Even if you ignore the fact that we weight some fields (title, author) much higher than others, a fundamental feature of DisMax is that it basically gives ranking based on the question, "What percentage of the words in the document match one of our query terms"?

Most of the time, that's exactly what you want. In general, items that have all the keywords, and more of them, appear at the top of the search results.

But sometimes you can have just, say, two of your three search terms appearing like a rash all across a relatively short record, and it'll pop to the top, appearing ahead of records that actually contain all three search terms. Or maybe three of four search terms appear in both title and author (highly-weighted fields) and the same thing happens.

And, yeah, it really happens.

### An actual, real-life example

Searching for the three terms [information AND architecture AND usability](http://mirlyn.lib.umich.edu/Search/Home?inst=all&amp;lookfor=information+AND+architecture+AND+usability), explicitly requiring all three, gives 12 results.

The [equivalent DisMax search](http://mirlyn.lib.umich.edu/Search/Home?inst=all&amp;lookfor=information+architecture+usability) (where only two of three need to be found) nets about 4300 results. Which is great -- we're casting a much wider net, with some pretty common words. That doesn't matter so long as the most relevant results float to the top.

The kicker? The first time an item in the first set appears in the second is at record number **62**. Our user is more than three pages in before she even see a record that contains all three terms.

Again, most of the time, our current algorithm does really, really well in my opinion. But noticing this led to talk about artificially pushing all the "all terms are present" items to the top.

### Pushing records that contain all the terms to the top

So, I wanted to:

* Push records with all search terms to the top, but
* ...don't otherwise change their scores. i.e., don't otherwise re-order them in any way, 'cause I'm already happy with my ordering.

It turns out to be harder than I initially thought. I fought with my code for a whole day, then asked for help, and [help was provided](http://www.lucidimagination.com/search/document/1f2e58ecc9c16a9c/function_query_to_boost_scores_by_a_constant_if_all_terms_are_present).

So, with special thanks to Jan Høydahl for his solution, we get this, in Ruby psuedocode:

andedTerms = allMyTerms.join(' AND ')
bf = map(query($qq),0,0,0,100000.0) # Add this value to the ranking score
qq = "allFields:(#{andedTerms})" # Use this as the query
## add bf and qq to your solr query

The `qq` is easy enough -- it basically says that to get any relevancy score at all, the record must have all the terms in the `allFields` Solr field.

For the `map`, we want to say
&gt; If the record matches all the terms, give it an extra 100K points. If not, don't.

The `map` takes 5 arguments:

* An initial value. In this case, we're getting the relevancy ranking score based on the `qq` query. Basically, items that don't have all the terms will have a score of zero; items that *do* have all three terms will have something bigger than zero.
* The beginning of range to compare to. In this case, 0.
* The end of the range. Another zero, so basically, we'll be seeing if our initial value is between 0 and 0, e.g., if it's exactly 0.
* The value to return if the initial value fits in the range -- zero. So, if the records doesn't have all the terms, return a 0.
* The value to return if the initial value falls outside the given range. 100K -- a random very-large number I picked.

### And...?

I just pushed this to our beta site, and folks are still looking at it, but so far, it looks awesome. I'll do a little update post if/when it goes into production. And if it doesn't, I'll say why.
