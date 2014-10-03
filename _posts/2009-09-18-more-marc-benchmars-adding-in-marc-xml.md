---
title: "More Ruby MARC Benchmarks: Adding in MARC-XML"
date: 2009-09-18
layout: post

---

It turns out that UVA's reluctance to use the raw MARC data on the search results screen is driven more by processing time than parsing time. Even if they were to start with a fully-parsed MARC object, they're doing enough screwing around with that data that the bottleneck on their end appears to be all the regex and string processing, not the parsing. Their specs for what gets displayed are complex enough that they want to do the work up-front.

But I remain interested, at least partially because of the reason UVA is using MARC-XML: they have MARC records too big for binary MARC format to handle. We do, too, and we've just been talking about what to do with them. So I'm thinking that

First, I spent some time dusting off my first attempt at ruby programming: modifying ruby-marc to use libxml if it's available. It's not super-well tested, but I'm pretty sure it works. And the speed increases are ... well, see below.

Anyone who wants to mess with my [attempt at libxml-enabled ruby-marc](http://github.com/billdueber/BillDueber-ruby-marc) is welcome to do so. This is a *very* forgiving parser -- it trusts that whatever ended up in the XML should, in fact, have been there. If you say 'XXE' is a control field, well, I'll treat it as a control field.

But back to the data. A few points are obvious:

* XML with REXML is dead-slow on both platforms (at least an order of magnitude slower )
* XML with LibXML is competitive with binary MARC (within 20% or so)
* Even with REXML, though, time to create MARC records out of the 50 input strings is less than a second, which might be ok depending on your application.


### Full results

As with last time, the *total* numbers below show how long it took to process all 40 sets of 50 records. The unadorned numbers are the *average time it took to process a set of 50 records*.

#### Call up solr with a null search, get 2000 records back in batches of 50 with wt=ruby, eval it, and stick it into arrays

    jruby-Get/Eval data              0.143550
    mri-Get/Eval data                0.106550

    jruby-Get/Eval data (total)      5.742000
    mri-Get/Eval data (total)        4.262017


#### Turn raw strings into MARC::Record objects from MARC-Binary strings, joining all the returned MARC together first
    jruby-marc4j-multistring         0.026575
    jruby-marc-multistring           0.037175
    mri-marc-multistring             0.073396

    jruby-marc4j-multistring (total) 1.063000
    jruby-marc-multistring (total)   1.487000
    mri-marc-multistring (total)     2.935842

#### Turn raw strings into MARC::Record objects from MARC-XML
    mri-marc-LibXML                  0.091332
    jruby-marc-REXML                 0.799500
    mri-marc-REXML                   0.948549

    mri-marc-LibXML (total)          3.653276
    jruby-marc-REXML (total)        31.980000
    mri-marc-REXML (total)          37.941975

### Conclusions

I'm not sure exactly where this leaves me, other than knowing that marc-xml is probably a viable alternative if you can use libxml. Getting a version of that code which uses native Java XML libraries when run under jruby  might be a useful exercise.
