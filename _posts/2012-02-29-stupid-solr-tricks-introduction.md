---
title: "Stupid Solr tricks: Introduction (SST #0)"
date: 2012-02-29
tags: "solr, Stupid Solr Tricks"
layout: post

---

**Completed parts of the series:**

1. [A Solr Field Type for numeric(ish) IDs](http://robotlibrarian.billdueber.com/solr-field-type-for-numericish-ids/)
2. [Using localparams in Solr (or, how to boost records that contain all terms)](http://robotlibrarian.billdueber.com/using-localparams-in-solr-sst-2/)
3. [Requiring/Preferring searches that don't span multiple values](http://robotlibrarian.billdueber.com/requiringpreferring-searches-that-dont-span-multiple-values-sst-3/)
4. [Boosting on Exactish (anchored) phrase matching](http://robotlibrarian.billdueber.com/boosting-on-exactish-anchored-phrase-matching-in-solr-sst-4/)

Those of you who read this blog regularly (Hi Mom!) know that while we do a lot of stuff at the [University of Michigan Library](http://lib.umich.edu), our bread-and-butter these days are projects that center around [Solr](http://lucene.apache.org/solr/).

Right now, my production Solr is running an ancient nightly of version 1.4 (i.e., before 1.4 was even officially released), and reflects how much I didn't know when we first started down this path. My primary responsibility is for [Mirlyn](http://mirlyn.lib.umich.edu), our catalog, but there's plenty of smart people doing smart things around here, and I'd like to be one of them.

Solr has since advanced to 3.x (with version 4 on the horizon), and during that time I've learned a lot more about Solr and how to push it around. More importantly, I've learned a _lot_ more about our data, the vagaries in the MARC/AACR2 that I process and how awful so much of it really is.

So...starting today I'm going to be doing some on-the-blog experiments with a new version of Solr, reflecting some of the problems I've run into and ways I think we can get more out of Solr.

### Premise 1: put all the logic you possible can into Solr

Much of what I'll be doing is looking at new field type definitions that are appropriate (in my mind, anyway) for library data. Some of this stuff (e.g., normalizing ISBNs) would be a *lot* easier to do in your indexing code.

But then you'd have to do it again in your application to munge whatever is entered in the search box. And maybe it won't be the same every time. Or maybe you don't want to write a freakin' parser to try to find anything that might look like an ISBN and mess with it.

I take it as gospel that you should put all your logic into the solr field analysis chain, so the exact same thing is happening on index and on query. That way, even if it's *wrong*, at least it'll be wrong in the exact same way and your users will find the stuff they're looking for.

### Premise 2: Doing it crappily is better than not doing it at all.

Look, the _right_ way to do much of this stuff is by hacking on Solr itself, building custom field analyzers or filters or tokenizers that mess with the token chain and...

Wait. I already lost myself, and probably you, too. At some point, I'm going to do an actual sample custom filter for the new Solr codebase (the stuff I did once before is out-of-date); the example will be LCCN normalization and you'll be able to follow along with me on this blog.

But in the meantime, we can do a lot of fairly ambitious stuff just by using and abusing the out-of-the-box stuff: pattern replacement filters, the existing tokenizers, etc. It might be ugly, and not very fast, but if I start getting the 200 hits a second that mean this is a bottleneck for me, I'll be happy to deal with it then.

### Premise 3: It's always better to put _something_ out there so smart people can tell you how to do it _right_

One of the disappointments in my life right now is that there isn't more formal and informal discussion about what people are doing/trying. I'm sure it's out there, but some of it is buried in a sea of application-level crap, and much of it is ignored by the people that really understand the data.

With luck, I'll get comments from folks who really know their stuff and can tell me, in excruciating detail, exactly how I _don't_. Please: correct me. I might not be the brightest guy in the room, but I know enough to try to outsource my thinking.

### Follow along at home!

#### Option 1: Build your own current-trunk Solr

If you want to follow along at home, you'll need a copy of the current source (not the 3.5 stable, since I use things like the ICUTokenizer coming in 3.6 / 4.0), which you can find and build from the Solr site.

#### Option 2: Just use what I'm using

Alternately, if you're lazy (and who isn't??), I've provided a [github repo](https://billdueber@github.com/billdueber/solr_stupid_tricks) of the standard solr "example" directory you can nab and run on your own java-equipped machine.

**Warning: the git repo is currently 60MB or so**.

~~~bash
  git clone https://billdueber@github.com/billdueber/solr_stupid_tricks.git
  cd solr_stupid_tricks
  java -jar start.jar
~~~
...and then head to your local [Solr Admin page](http://localhost:8983/solr/admin/) page on port 8983 to check things out. We'll be spending most of our time in [the analysis tab](http://localhost:8983/solr/admin/analysis.jsp).

I'll get the first post in the series up later today, and then every few days as I think of more things to talk about. I hope you'll join me!
