---
title: "Pushing MARC to Solr; processing times and threading and such"
date: 2010-03-04
---

[This is in response to a [thread on the blacklight mailing](http://groups.google.com/group/blacklight-development/browse_thread/thread/672b7269ada16a61?hl=en) list about getting MARC data into Solr.]

### What's the question?

The question came up, "How much time do we spend processing the MARC vs trying to push it into Solr?". Bob Haschart found that even with a pretty damn complicated processing stage, pushing the data to solr was still, at best,
taking at least as long as the processing stage. 

I'm interested because I've been struggling to write a solrmarc-like system that runs under JRuby. Architecturally, the big difference between my stuff and solrmac is that I use the StreamingUpdateSolrServer (on Erik Hatcher's suggestion). So I thought I'd check how things break down for me.

Here are my numbers running under JRuby (using MARC4J as the marc
implementation) with the Solr StreamingUpdateSolrServer. Obviously, there are
a lot of differences between this and solrmarc, but I'm hoping that while it's
not comparing apples to apples, it's at least comparing apples to some sort of
processed cheese-like product.

### What work is being done on what?

The data set is a file of 18,881 MARC records in marc-binary format. It's
probably not big enough to get a great idea of how things will run over the
long (many millions of records) haul, but it'll do for this rough-cut stuff.

I break my processing down into five categories:

* Read the records into marc4j objects and do nothing. This is a baseline of sorts.
* The "normal" fields are anything that you could do with SolrMarc without a
custom routine; the actual processing is done in JRuby. 
* Custom fields are generated with JRuby code, but these are things that in solmarc would require a custom routine.
* The big "allfields" field is text from tags 100 through 900.
* The "to_xml" routine is just calling the underlying marc4j XML output and stuffing it into a string.

The schema used is our normal UMICH schema *except for* High Level Browse
(which appear in the [our catalog](http://mirlyn.lib.umich.edu/) as "Academic
Discipline"). The code for that is written in Java, and I just call it from
JRuby when I'm using it. I excluded it because it's incredibly expensive, both at startup time (when it loads a giant database of call-number ranges and associated categories) and for processing -- there's a lot of call-number normalization, long-string comparisons, some modified binary searches, etc. etc. etc. It's expensive. Trust me. 

The Solr server itself is on a different, incredibly-beefy machine, and is
emptied out before each invocation that involves actually pushing data to it (with a delete-by-query *:*).

### How fast were things on my desktop?

* 18,881 records in marc-binary format
* Times are in seconds, run on my desktop
* Remember, you can't compare these numbers to Bob's because we're doing
different things to different data. 

|Total Seconds  | Description |
|------- | -------------- |
| 19 | Just read the records with marc4j and do nothing.|
| 85 | Read and do 35 "normal" fields (no custom)|
|104 | Read, 35 normal, 15 custom fields|
|110 | Read, normal, custom, allfields|
|129 | Read, normal, custom, allfields, to_xml|
|136 | Read, normal, custom, allfields, to_xml, 2-threaded SUSS, commit every 5K docs|
|142 | Read, normal, custom, allfields, to_xml, 1-threaded SUSS, commit every 5k docs|
|124 |Read, normal, custom, allfields, to_xmx, 1-threaded SUSS, commit every 5k docs, **2 threads doing processing**|

We can also break the same numbers down as:

Seconds | Description
-------- | -----------
  19 | read the records and do nothing
  66 | process the 35 normal fields
  19 | process the 15 custom fields
   6 | generate the "allfields" field
  19 | generate the XML (yowza!) 
   7 | send to solr with two threads
  13 | send to solr with one thread

Or like this:

Seconds | Description
--------| -----------
 129 | do all the reading and processing
  13 | send to solr with one thread

### Why does solr processing seem so much faster for me?

There are a lot of reasons why my submit-to-solr might seem like less of a
burden. The ones I can think of off the top of my head are:

* SUSS is just faster than whatever solrmarc does. 
* My processing stage is so much slower than solrmac's (due to algorithms or jruby-vs-java, I don't know) that the "push to solr" portion of it gets swallowed up by the slowness of the of overall code.
* The Solr server is so much faster than my desktop that my poor little 
  desktop can't send it data fast enough to work it.

**For my setup, obviously adding a processing thread is a lot more beneficial
than adding a SUSS thread.** My desktop doesn't have that many threads lying around (adding a third processing thread actually slowed things down), so I moved the code to a beefier machine to see what happened.

### Trying the same thing on a beefy machine

This is the exact same code and data, but on a beefy machine (16 cores, gobs
of memory).

time   |  SUSS Threads  |   Processing Threads
-----: | :--------------: | :-------------------:
70    |    1      |    1     (was 142 seconds on the desktop)
47    |    1      |    2
39    |    1      |    3
35    |    1      |    4
68    |    2      |    1
48    |    2      |    2
38    |    2      |    3
34    |    2      |    4

So, on my hardware anyway, there's a sweet spot with one suss thread and
three processing threads. YMMV, of course.

### What have we learned?

I'm not sure, to be honest. It's logistically difficult for me to do the same
process in solrmarc because I'd have to rebuild everything without the HLB stuff. I guess for me, what I've learned that if I'm going to continue working 
on my code, the places to focus my attention are threading (obviously) and MARC-XML generation.
