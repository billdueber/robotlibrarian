---
title: "Benchmarking MARC record parsing in Ruby"
date: 2009-09-17
---

[Note: since I started writing this, I found out Bess & Co. store MARC-XML. That makes a difference, since XML in Ruby can be really, really slow]

[**UPADTE** It turns out they *don't* use MARC-XML. They use MARC-Binary just like the rest of us. Oops. ]

[**UP-UPDATE** Well, no, they *do* use MARC-XML. I'm not afraid to constantly change my story. This is why I'm the best investigative reporter in the business]

The other day on the blacklight mailing list, Bess Sadler wrote

> Yes, we do still include the full marc record, but the rule of thumb we're currently using is that anything that needs to display in the index view (the search results) needs to be broken out into a separate display field, because retrieving and parsing marc records for every item in a list of search results is too much of a performance hit.

This surprised me a fair bit, because in our implementation of VuFind (which uses PHP, versus Ruby for Blacklight) I do just that -- grab the MARC out of Solr, parse it, and pull stuff like full titles and such out of it. 

As it turns out, I'd been screwing around with calling marc4j from jruby, anyway, so I threw that into the mix, and here's what I found.

### What the benchmark tries to measure

The focus is on measuring time to parse MARC records as returned in a field from Solr in MARC-binary.

I got 40 sets of 50 records each (2000 records) from our Solr instance in ruby format and extracted the binary MARC strings. This resulted in an array of 40 sets of 50 strings, each of which is a valid MARC record. 

Fifty records seems largish to me -- we only display 20 at a time -- but thought I'd swing for the fences.

I'm testing along three(ish) dimensions:

* jruby vs mri
* marc4j vs ruby-marc (only on jruby, obviously)
* parsing each string individually, or globbing them all together and treating it as if it's a multi-record file

[Note that MRI is using Net::HTTP to get the data; I presume Curl would be faster still. It's already faster than jruby]

The following data show the average time to parse out each set of 50 records and extract the first 245 (title) field from each one, along with the totals for doing all 2000 records.

    Method                           User       Total      Real      
    
    jruby Get/Eval data              0.134750   0.134750 (  0.134850)
    jruby Get/Eval data (2000)       5.390000   5.390000 (  5.394000)

    MRI Get/Eval data                0.008500   0.012750 (  0.115942)
    MRI Get/Eval data (2000)         0.340000   0.510000 (  4.637677)    
    
    jruby-marc4j-oneAtATime          0.056075   0.056075  (0.056125)
    jruby-marc4j-multistring         0.027925   0.027925  (0.028000)
                                                          
    jruby-marc-oneAtATime            0.066625   0.066625  (0.066650)
    jruby-marc-multistring           0.034300   0.034300  (0.034325)
                                                          
    mri-marc-oneAtATime              0.084500   0.085250  (0.086597)
    mri-marc-multistring             0.085000   0.085750  (0.086026)
                                                          
    jruby-marc4j-oneAtATime (2000)   2.243000   2.243000  (2.244999)
    jruby-marc-oneAtATime (2000)     2.665001   2.665001  (2.666000)
    mri-marc-oneAtATime (2000)       3.380000   3.410000  (3.463888)
                                                          
                                                          
    jruby-marc4j-multistring (2000)  1.117001   1.117001  (1.120001)
    jruby-marc-multistring (2000)    1.371999   1.371999  (1.372999)
    mri-marc-multistring (2000)      3.400000   3.430000  (3.441052)
    

So...the worst-case scenario is taking an average 0.085 second to get the first title field out of **each one of 50 binary MARC records** once we've got them.

Now, I'm sure all my records came out of the cache, so my query time wasn't very long. But we still end up with a maximum of roughly 0.2 seconds plus the time to actually do the query to end up with a set of 50 marc records.

We can see from looking at the totals that it looks like MRI's bottleneck is 
the actual parsing, whereas constructing the input streams is expensive under jruby (at least the way I'm doing it), resulting in a benefit of concatenating them all together into one longish string before parsing.

Marc4j is faster (20%ish), but not enough faster to be worth the effort in my mind. Keep in mind that I have no idea how fast Marc4j is when running under pure java, without all the jruby overhead. 

Bottom line, though: that seems fast enough to me.

I'll try to benchmark with XML later on today or tomorrow.
