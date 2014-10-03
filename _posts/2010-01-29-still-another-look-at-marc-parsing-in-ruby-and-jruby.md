---
title: "Still another look at MARC parsing in ruby and jruby"
date: 2010-01-29
layout: post
slug: still-another-look-at-marc-parsing-in-ruby-and-jruby

---

I've been looking at making a jruby-based solr indexer for MARC documents, and started off wanting to make sure I could determine if anything I did would be faster than our existing (solrmarc-based) setup.


* <strong>Assertion</strong>: The upper bound on how fast I can process records and send them to Solr can be approximated by looking at how fast I can parse (and do nothing else to) marc records from a file.
* <strong>Assertion</strong>: If I can't write a system that's faster than what we have now, it's probably not worth my time even though being able to fall back to ruby instead of java would be nice.
* <strong>The Big Question</strong>: Is the MARC parsing process fast enough that it seems I might be able to write a system that runs faster than the solrmarc setup I have now?
* <strong>The Answer (see below)</strong>: Yes, if I use marc4j.

On our ridiculously-awesome hardware, right now we're doing about 300 records/second for short files and 250 records/second for a full (6.5 million record) index, giving us a 7-8 hour reindex.

I'll just post the results without a lot of commentary. I warmed stuff up in all cases, and ran on my desktop (so I could compare to MRI ruby, which isn't installed on the server) and on the server where we usually run these things.

* <em>The machines</em> are my desktop OSX machine and the beefy linux server where we usually do this stuff
* <em>The platforms</em> are jruby 1.4 --server and MRI ruby 1.87
* <em>The libraries</em> are marc4j and ruby-marc 0.3.3
* <em>The parsers</em> are

    * The standard binary parsers all around
    * A home-grown AlephSequential format reader for the 'seq' type. AlephSequential is a MARC representation that uses one line for each field. We use it because it doesn't have length limitations and, not surprisingly, Aleph can spit it out pretty quickly compared to MARC-XML.
    * Whatever marc4j uses internally for MARC-XML
    * ruby-marc's 'jstax' xml parser under jruby (which I wrote and apparently needs some love, see below)
    * ruby-marc's 'libxml' xml parser under MRI ruby

* <em>Seconds</em> is the average of two rounds, with measurements taken after a warmup run in each case.

The test files were 18,881 records in marc-xml, marc-binary, and AlephSequential formats.

~~~

MACHINE  PLATFORM LIBRARY     PARSER    SECONDS    REC/SECOND
desktop  jruby    marc4j      binary      4.06       4650
desktop  jruby    marc4j      xml         5.55       3401
desktop  jruby    ruby-marc   binary     17.35       1088
desktop  jruby    ruby-marc   jstax      80.11        236

desktop  ruby     ruby-marc   binary     33.54        562
desktop  ruby     ruby-marc   libxml     46.87        402

server   jruby    marc4j      binary      2.29       8245
server   jruby    marc4j      xml         3.36       5619
server   jruby    marc4j      AlephSeq    3.68       5130
server   jruby    ruby-marc   binary      9.93       1901
server   jruby    ruby-marc   jstax      44.56        424


~~~~


The quick takeaways, with all the obvious caveats:

* jruby with ruby-marc is twice as fast at binary and twice as slow at xml compared with MRI
* marc4j is four times as fast for binary and about an order of magnitutde faster for xml compared with ruby-marc.
* The server is fast.

We know from previous experience that libxml is the fastest of the current MRI-based marc-xml readers and that jstax is the best of the current jruby-based marc-xml readers. And, finally, we know that many of us can't use marc-binary format because our records are too big.

If I'm gonna use jruby (which I think I am due to wanting to use the StreamingUpdateSolrServer) I'm gonna need to use marc4j and just wrap it up in some nicer syntax.
