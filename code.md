---
layout: page
permalink: /code/
title: My code
---

[My github page](http://github.com/billdueber/) holds just about everything I've worked on, past and present, finished and incomplete, which makes it a bit of a mess.

The things I think are ready for prime-time, though, are as follows. Everything is in ruby, unless otherwise noted.

## Traject

[traject](https://github.com/traject-project/traject) (with Jonathan Rochkind) is an ETL (extract, transform, load) system with plenty of generic hooks but designed around the idea of indexing MARC records into Solr. I use this for both the University of Michigan catalog ([Mirlyn](http://mirlyn.lib.umich.edu)) and the [Hathitrust catalog](http://www.hathitrust.org/).

I've also put together a [sample configuration](https://github.com/traject-project/traject_sample) and make available the [full, nasty configuration used for Mirlyn and the HT catalog](https://github.com/billdueber/ht_traject).

We're interested in getting other folks using traject so we can build up an ecosystem of macros and scripts; don't be afraid to contact me if you're interestred.

### Associated work:

* [traject-alephsequential-reader](https://github.com/traject-project/traject_alephsequential_reader): A reader for the Ex Libris AlephSequential MARC serialization.
* [traject_umich_format](https://github.com/billdueber/traject_umich_format): A traject macro that implements the University of Michigan's process for determining the format of an item represented by a MARC record (e.g., book, serial, etc.).

## MARC

Given how much MARC I have to work with, it was inevitable, perhaps, that I would end up writing libraries to deal with it.

* [ruby-marc](https://github.com/ruby-marc/ruby-marc) (with many, many others): I've had a small part in improving and adapting this over the years, mostly focused on JRuby support and some speed improvements (because when you've got 10M bibs to index, speed counts!)
* [marc4j_extra_reader_writers](https://github.com/billdueber/marc4j_extra_reader_writers): Before [traject](https://github.com/traject-project/traject), and before I wrote my own indexing code, I used to use [solrmarc](https://code.google.com/p/solrmarc/) and, along with it, [marc4j](https://github.com/marc4j/marc4j). This implements a [marc-in-json](http://dilettantes.code4lib.org/blog/2010/09/a-proposal-to-serialize-marc-in-json/) reader/writer and an alephsequential reader for marc4j (in Java). **Note:** My own proposed JSON serialization for MARC [marc-hash](http://robotlibrarian.billdueber.com/2010/02/new-interest-in-marc-hash-json/) should be considered dead: the commuity has all gotten behind MARC-in-JSON.
* [ruby-marc-marc4j](https://github.com/billdueber/ruby-marc-marc4j): A simple gem to translate from a marc4j record (pulled in via JRuby with the marc4j .jar file) into a ruby-marc ruby object.
* [MARC::File::MiJ](http://search.cpan.org/~gmcharlt/MARC-File-MiJ/lib/MARC/File/MiJ.pm): I also wrote a perl implementation of marc-in-json for the MARC::Record set of modules (in Perl)
* [marc_alephsequential](http://github.com/billdueber/marc_alephsequential): An alephsequential reader for ruby-marc.

## Other library data

* [library_stdnums](https://github.com/billdueber/library_stdnums): Ruby code to identify, validate, and normalize ISBNs, ISSN, and LCCNs.

## General

* [threach](https://github.com/billdueber/threach) and [jruby_threach](https://github.com/billdueber/jruby_threach): Add a `#threach` method
to any Enumerable ruby object, allowing you to run a block across many threads at once. These days I'd probably just use a thread-pool from the [concurrent-ruby](https://github.com/ruby-concurrency/concurrent-ruby) project, but plenty of folks still use this in the wild.
* [match_map](https://github.com/billdueber/match_map): an object with a hash-like interface that allows many-to-many relationships. Keys are strings or regexes; values can be single or multiple, and many keys can match in a single lookup. Useful when you need it, confusing when you don't.
