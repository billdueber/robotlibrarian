---
title: "Using localparams in Solr (or, how to boost records that contain all terms) (SST #2)"
date: 2012-03-06
tags: "solr, Stupid Solr Tricks"
---

[Note: this isn't so much a _Stupid Solr Trick_ as a _Thing You Should Probably Know_; consider it required reading for the next SST. If you're just joining us, check out the [introduction to the Stupid Solr Tricks series](http://robotlibrarian.billdueber.com/stupid-solr-tricks-introduction/)]

### What the heck is a localparams query?

A garden-variety Solr query URL looks something like this:

~~~
  http://localhost:8983/solr/select?
    defType=dismax
    &amp;qf=name^2 place^1
    &amp;q=Dueber
~~~~

Which is fine, as far as it goes. But it's easy to run into the limits of the standard query plugins (e.g., Dismax).

Say, for example, you want something like this:

~~~
  title:Constructivism AND author:Dueber
~~~~

And furthermore, you have multiple underlying fields (title1, title2, title3, author1, author2).

The naïve approach would be to just do this:

~~~
  defType=dismax
  &amp;qf=title1 title2 title3 author1 author2
  &amp;q=Constructivism Dueber
~~~~

But you can't construct a dismax query with the boolean AND. You can with edismax, but even then you've got no way of telling (e)dismax that _Constructivism_ must be found in the title fields, and _Dueber_ must be found in the author fields. Dismax doesn't do that.

### Solution: Build a query of queries

The solution is to build a query made up of fully-encapsulated sub-queries. A localparams query has two forms (note that, of course, you'd need to URL-Escape the values):

~~~
  _query_:"{!dismax qf='field^2 otherfield^4'}my search terms"
or
  _query_:"{!dismax qf='field^2 otherfield^4' v=$q1}" & q1=my search terms
~~~~

I far prefer the second form (which uses a second URL parameter `q1` instead of sticking the search right in there), because I don't have to worry about escaping double-quotes in the query terms (as you would if there's a phrase as part of the query).

Once you've got these things, you can combine them with booleans.

~~~
    q=_query_:"{!dismax qf='title1 title2 title3' v=$q1}" AND
      _query_:"{!dismax qf='author1 author2' v=$q2}"
  &q1=Constructivism
  &q2=Dueber
~~~~

[Note: **[be careful with solr booleans!!!](http://robotlibrarian.billdueber.com/solr-and-boolean-operators/)**]

You can add any _local parameters_ you need (for dismax, stuff like mm, qs, pf, and ps) and you can use any query parser you want by changing what comes after the bang (e.g., `{!lucene ...}` or `{!edismax...}`).

In this way, you can build up arbitrarily complex queries using any available query parsers in combination with each other. Very powerful.

### An example: boost records that contain all terms

Just about everything in a localparams query can be pulled out in the way I pulled out the search terms above. Here's a fairly-complex example (which, let's be honest, would be a lot more complex if you were trying to inline and escape everything).

**Scenario**: We want to do a logical-OR search (mm=0%), but want to make sure we boost documents that contain all the search terms. This is necessary because sometimes a very long document with all the terms will have a lower score than a very short document with most of the terms.

Having short document with a few keywords show up before long documents with all the keywords will drive your librarians _CraZy_!!! So it's tempting to just leave it alone. But let's fix it anyway.

The gist of it is as follows:

* Query against title and author
* Use an mm of 0% (logical OR) for the main query
* Use a pf to boost on a phrase in those same two fields (just common sense)
* Set up a boost query (bq) to boost the score if *all* the search terms are present

To accomplish this, we're going to have two localparams queries: one to be the main query, and another that we're going to use as the boost query. This works in much the same way as our previous "AND-together two localparams queries" did.

[Presenting the URL parameters as a ruby hash to make it easier to read]

~~~ruby
  {
  'q'=>'_query_:"{!dismax qf=$f1 mm=$mm1 pf=$f1 bq=$bq1 v=$q1}"',
  'mm1'=>'0%',
  'f1'=>'author^3 title^1',
  'q1'=>'Dueber Constructivism',
  'bq1'=>'_query_:"{!dismax qf=$f1 mm=\'100%\' v=$q1 }"^5',
  'fl' =>; 'score,*'
  }
    
    
~~~
    
What's nice about this is that I'm reusing the search terms (for the main query and the boost query) and field list (for the query field and the phrase fields) so I don't have to repeat them.

### Try along at home

First off, if you don't have a browser that does nice XML and JSON formatting, well, get one. I use Chrome with [JSONView](https://chrome.google.com/webstore/detail/chklaanhfefbnpoihckbnefhakgolnmc) and [XMLTree](https://chrome.google.com/webstore/detail/gbammbheopgpmaagmckhpjbfgdfkpadb), but I'm sure there are equivalents for Firefox. They'll make your life easier.

By now you know the drill:

~~~bash
  cd solr_stupid_tricks
  git pull origin master
  git fetch origin master
  git checkout SST2 # I've started tagging the repo for these posts
  # ignore warning about "detached HEAD"
  java -jar start.jar &
~~~

We'll want to empty out the index and put in some documents to work with. I'm presuming you have `curl` installed. If not...well, you're on your own.

~~~bash
  cd exampledocs
  ./reset_and_index_json localparams.json
~~~

You might want to take a look at the `localparams.json` file, which contains a set of documents in the new JSON update structure. The [full Solr JSON Update structure](http://wiki.apache.org/solr/UpdateJSON) allows repeated keys. Apparently, so does the [JSON RFC](http://www.ietf.org/rfc/rfc4627):

> 2.2. Objects
> An object structure is represented as a pair of curly brackets
> surrounding zero or more name/value pairs (or members). A name is a
> string. A single colon comes after each name, separating the name
> from the value. A single comma separates a value from a following
> name. **The names within an object SHOULD be unique.** _(emphasis mine)_

"SHOULD". Not "MUST". I don't care if it's legal. It still weirds me out.

Once you've got solr running in the background, you can go ahead and try our query!

* If you're really lazy, just [click the link](http://localhost:8983/solr/select/?q=\_query\_%3A%22%7B%21dismax+qf%3D%24f1+mm%3D%24mm1+pf%3D%24f1+bq%3D%24bq1+v%3D%24q1%7D%22&amp;mm1=0%25&amp;f1=author%5E3+title%5E1&amp;q1=Dueber+Constructivism&amp;bq1=\_query\_%3A%22%7B%21dismax+qf%3D%24f1+mm%3D%27100%25%27+v%3D%24q1+%7D%22%5E5&amp;fl=score%2C%2A)
* If you're slightly less lazy, and you've got ruby installed, take a look in the new ruby directory. You can run `ruby browse.rb localparams_query.rb` to run the query and have it automatically open up in your browser.
* If you're ambitious, you might want to actually mess with the `localparams_query.rb` file so you can try things out.

As a longish side note, we'll probably use `browse.rb` in the future of this series as well, so you might want to go ahead and get ruby installed if you don't already. [RVM](https://rvm.beginrescueend.com/) is the easiest route if you're on linux/OSX. You can also just install [JRuby](http://jruby.org/), seeing as how you're running java anway (just make sure to use 1.9 mode by calling stuff as `jruby --1.9 myscript.rb` or setting the environment variable `export JRUBY_OPTS=--1.9`).

### Special Stupid Solr Trick: Make a special query handler for a complex query

OK, so I said I wouldn't have a real SST in this episode, but it's so damn long at this point I figure I've lost everyone except Rochkind (Hey, Jonathan!), so let's throw one in.

The Solr configuration file `solrconfig.xml` is where you can configure custom search handlers. In such a custom handler, you can specify defaults (which, by default, can be overridden by passed-in parameters, although you can control that, too) -- this is commonly used to, say, put in a `q.alt` or a filter query that will always be applied.

But we can use it to put in our special query defaults that boosts when a document contains all the terms:

~~~xml
  

      10
      *,score
      explicit

        _query_:"{!edismax qf=$fields
                           mm=$mymm
                           v=$qwords
                           bq=$boostForAll}"

      0%

        _query_:"{!edismax qf=$fields
                           mm='100%'
                           v=$qwords }"^5
~~~

If you look closely, you'll see that everything you need is defined in this requestHandler in the `solrconfig.xml` file, except for `$fields` and `$qwords`. You could also override `mymm` by passing in an argument with that name, if the default '0%' isn't to your liking.

If you've been following along at home, this requestHandler is already in the `solrconfig.xml` file that you're running right now. Go ahead and try it! Let's search for the terms 'dueberb' and 'penn' and see if the correct record floats to the top.

[http://localhost:8983/solr/edismaxplus/?qwords=dueber penn&amp;fields=author title](http://localhost:8983/solr/edismaxplus/?qwords=dueber%20penn&amp;fields=author%20title)

Nifty, huh?

Next time we'll use a local params query to get around something about dismax that drives me crazy: preventing (or penalizing) matches that go across a field's multiple values.100%
What's nice about this is that I'm reusing the search terms (for the main query and the boost query) and field list (for the query field and the phrase fields) so I don't have to repeat them.

### Try along at home

First off, if you don't have a browser that does nice XML and JSON formatting, well, get one. I use Chrome with [JSONView](https://chrome.google.com/webstore/detail/chklaanhfefbnpoihckbnefhakgolnmc) and [XMLTree](https://chrome.google.com/webstore/detail/gbammbheopgpmaagmckhpjbfgdfkpadb), but I'm sure there are equivalents for Firefox. They'll make your life easier.

By now you know the drill:

~~~bash
  cd solr_stupid_tricks
  git pull origin master
  git fetch origin master
  git checkout SST2 # I've started tagging the repo for these posts
  # ignore warning about "detached HEAD"
  java -jar start.jar &amp;
~~~
We'll want to empty out the index and put in some documents to work with. I'm presuming you have `curl` installed. If not...well, you're on your own.

~~~bash
  cd exampledocs
  ./reset_and_index_json localparams.json
~~~
You might want to take a look at the `localparams.json` file, which contains a set of documents in the new JSON update structure. The [full Solr JSON Update structure](http://wiki.apache.org/solr/UpdateJSON) allows repeated keys. Apparently, so does the [JSON RFC](http://www.ietf.org/rfc/rfc4627):

&gt; 2.2. Objects
&gt; An object structure is represented as a pair of curly brackets
&gt; surrounding zero or more name/value pairs (or members). A name is a
&gt; string. A single colon comes after each name, separating the name
&gt; from the value. A single comma separates a value from a following
&gt; name. **The names within an object SHOULD be unique.** _(emphasis mine)_

"SHOULD". Not "MUST". I don't care if it's legal. It still weirds me out.

Once you've got solr running in the background, you can go ahead and try our query!

* If you're really lazy, just [click the link](http://localhost:8983/solr/select/?q=\_query\_%3A%22%7B%21dismax+qf%3D%24f1+mm%3D%24mm1+pf%3D%24f1+bq%3D%24bq1+v%3D%24q1%7D%22&amp;mm1=0%25&amp;f1=author%5E3+title%5E1&amp;q1=Dueber+Constructivism&amp;bq1=\_query\_%3A%22%7B%21dismax+qf%3D%24f1+mm%3D%27100%25%27+v%3D%24q1+%7D%22%5E5&amp;fl=score%2C%2A)
* If you're slightly less lazy, and you've got ruby installed, take a look in the new ruby directory. You can run `ruby browse.rb localparams_query.rb` to run the query and have it automatically open up in your browser.
* If you're ambitious, you might want to actually mess with the `localparams_query.rb` file so you can try things out.

As a longish side note, we'll probably use `browse.rb` in the future of this series as well, so you might want to go ahead and get ruby installed if you don't already. [RVM](https://rvm.beginrescueend.com/) is the easiest route if you're on linux/OSX. You can also just install [JRuby](http://jruby.org/), seeing as how you're running java anway (just make sure to use 1.9 mode by calling stuff as `jruby --1.9 myscript.rb` or setting the environment variable `export JRUBY_OPTS=--1.9`).

### Special Stupid Solr Trick: Make a special query handler for a complex query

OK, so I said I wouldn't have a real SST in this episode, but it's so damn long at this point I figure I've lost everyone except Rochkind (Hey, Jonathan!), so let's throw one in.

The Solr configuration file `solrconfig.xml` is where you can configure custom search handlers. In such a custom handler, you can specify defaults (which, by default, can be overridden by passed-in parameters, although you can control that, too) -- this is commonly used to, say, put in a `q.alt` or a filter query that will always be applied.

But we can use it to put in our special query defaults that boosts when a document contains all the terms:

~~~xml
  

      10
      *,score
      explicit

        _query_:"{!edismax qf=$fields
                           mm=$mymm
                           v=$qwords
                           bq=$boostForAll}"

      0%

        _query_:"{!edismax qf=$fields
                           mm='100%'
                           v=$qwords }"^5
~~~
If you look closely, you'll see that everything you need is defined in this requestHandler in the `solrconfig.xml` file, except for `$fields` and `$qwords`. You could also override `mymm` by passing in an argument with that name, if the default '0%' isn't to your liking.

If you've been following along at home, this requestHandler is already in the `solrconfig.xml` file that you're running right now. Go ahead and try it! Let's search for the terms 'dueberb' and 'penn' and see if the correct record floats to the top.

[http://localhost:8983/solr/edismaxplus/?qwords=dueber penn&amp;fields=author title](http://localhost:8983/solr/edismaxplus/?qwords=dueber%20penn&amp;fields=author%20title)

Nifty, huh?

Next time we'll use a local params query to get around something about dismax that drives me crazy: preventing (or penalizing) matches that go across a field's multiple values. v=$q1 }"^5',
  'fl' =&gt; 'score,*'
  }
~~~
What's nice about this is that I'm reusing the search terms (for the main query and the boost query) and field list (for the query field and the phrase fields) so I don't have to repeat them.

### Try along at home

First off, if you don't have a browser that does nice XML and JSON formatting, well, get one. I use Chrome with [JSONView](https://chrome.google.com/webstore/detail/chklaanhfefbnpoihckbnefhakgolnmc) and [XMLTree](https://chrome.google.com/webstore/detail/gbammbheopgpmaagmckhpjbfgdfkpadb), but I'm sure there are equivalents for Firefox. They'll make your life easier.

By now you know the drill:

~~~bash
  cd solr_stupid_tricks
  git pull origin master
  git fetch origin master
  git checkout SST2 # I've started tagging the repo for these posts
  # ignore warning about "detached HEAD"
  java -jar start.jar &amp;
~~~
We'll want to empty out the index and put in some documents to work with. I'm presuming you have `curl` installed. If not...well, you're on your own.

~~~bash
  cd exampledocs
  ./reset_and_index_json localparams.json
~~~
You might want to take a look at the `localparams.json` file, which contains a set of documents in the new JSON update structure. The [full Solr JSON Update structure](http://wiki.apache.org/solr/UpdateJSON) allows repeated keys. Apparently, so does the [JSON RFC](http://www.ietf.org/rfc/rfc4627):

&gt; 2.2. Objects
&gt; An object structure is represented as a pair of curly brackets
&gt; surrounding zero or more name/value pairs (or members). A name is a
&gt; string. A single colon comes after each name, separating the name
&gt; from the value. A single comma separates a value from a following
&gt; name. **The names within an object SHOULD be unique.** _(emphasis mine)_

"SHOULD". Not "MUST". I don't care if it's legal. It still weirds me out.

Once you've got solr running in the background, you can go ahead and try our query!

* If you're really lazy, just [click the link](http://localhost:8983/solr/select/?q=\_query\_%3A%22%7B%21dismax+qf%3D%24f1+mm%3D%24mm1+pf%3D%24f1+bq%3D%24bq1+v%3D%24q1%7D%22&amp;mm1=0%25&amp;f1=author%5E3+title%5E1&amp;q1=Dueber+Constructivism&amp;bq1=\_query\_%3A%22%7B%21dismax+qf%3D%24f1+mm%3D%27100%25%27+v%3D%24q1+%7D%22%5E5&amp;fl=score%2C%2A)
* If you're slightly less lazy, and you've got ruby installed, take a look in the new ruby directory. You can run `ruby browse.rb localparams_query.rb` to run the query and have it automatically open up in your browser.
* If you're ambitious, you might want to actually mess with the `localparams_query.rb` file so you can try things out.

As a longish side note, we'll probably use `browse.rb` in the future of this series as well, so you might want to go ahead and get ruby installed if you don't already. [RVM](https://rvm.beginrescueend.com/) is the easiest route if you're on linux/OSX. You can also just install [JRuby](http://jruby.org/), seeing as how you're running java anway (just make sure to use 1.9 mode by calling stuff as `jruby --1.9 myscript.rb` or setting the environment variable `export JRUBY_OPTS=--1.9`).

### Special Stupid Solr Trick: Make a special query handler for a complex query

OK, so I said I wouldn't have a real SST in this episode, but it's so damn long at this point I figure I've lost everyone except Rochkind (Hey, Jonathan!), so let's throw one in.

The Solr configuration file `solrconfig.xml` is where you can configure custom search handlers. In such a custom handler, you can specify defaults (which, by default, can be overridden by passed-in parameters, although you can control that, too) -- this is commonly used to, say, put in a `q.alt` or a filter query that will always be applied.

But we can use it to put in our special query defaults that boosts when a document contains all the terms:

~~~xml
  

      10
      *,score
      explicit

        _query_:"{!edismax qf=$fields
                           mm=$mymm
                           v=$qwords
                           bq=$boostForAll}"

      0%

        _query_:"{!edismax qf=$fields
                           mm='100%'
                           v=$qwords }"^5
~~~
If you look closely, you'll see that everything you need is defined in this requestHandler in the `solrconfig.xml` file, except for `$fields` and `$qwords`. You could also override `mymm` by passing in an argument with that name, if the default '0%' isn't to your liking.

If you've been following along at home, this requestHandler is already in the `solrconfig.xml` file that you're running right now. Go ahead and try it! Let's search for the terms 'dueberb' and 'penn' and see if the correct record floats to the top.

[http://localhost:8983/solr/edismaxplus/?qwords=dueber penn&amp;fields=author title](http://localhost:8983/solr/edismaxplus/?qwords=dueber%20penn&amp;fields=author%20title)

Nifty, huh?

Next time we'll use a local params query to get around something about dismax that drives me crazy: preventing (or penalizing) matches that go across a field's multiple values.
