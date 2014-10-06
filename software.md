---
layout: page
permalink: /code/
title: Code
---

[My github page](http://github.com/billdueber/) holds just about everything I've worked on, past and present, finished and incomplete, which makes it a bit of a mess.

The things I think are ready for prime-time, though, are as follows. Everything is in ruby, unless otherwise noted.

## Traject

* [traject](https://github.com/traject-project/traject) (with Jonathan Rochkind): an ETL (extract, transform, load) with plenty of generic hooks but designed around the idea of indexing MARC records into Solr. I use this for both the University of Michigan catalog ([Mirlyn](http://mirlyn.lib.umich.edu)) and the [Hathitrust catalog](http://www.hathitrust.org/).

I've also put together a [sample configuration](https://github.com/traject-project/traject_sample) and make available the [full, nasty configuration used for Mirlyn and the HT catalog](https://github.com/billdueber/ht_traject).

### Associated work:

* [traject-alephsequential-reader](https://github.com/traject-project/traject_alephsequential_reader): A reader for the Ex Libris AlephSequential MARC serialization.
* [traject_umich_format](https://github.com/billdueber/traject_umich_format): A traject macro that implements the University of Michigan's process for determining the format of an item represented by a MARC record (e.g., book, serial, etc.).


## MARC

Given how much MARC I have to work with, it was inevitable, perhaps, that I would end up writing libraries to deal with it.

* [ruby-marc](https://github.com/ruby-marc/ruby-marc) (a small part helping many, many others): I've had a small part in improving and adapting this over the years, mostly focused on JRuby support and some speed improvements (because when you've got 10M bibs to index, speed counts!)
* [marc4j_extra_reader_writers](https://github.com/billdueber/marc4j_extra_reader_writers): Before [traject](https://github.com/traject-project/traject), and before I wrote my own indexing code, I used to use [solrmarc](https://code.google.com/p/solrmarc/) and, along with it, [marc4j](https://github.com/marc4j/marc4j). This implements a [marc-in-json](http://dilettantes.code4lib.org/blog/2010/09/a-proposal-to-serialize-marc-in-json/) reader/writer and an alephsequential reader for marc4j (in Java). **Note:** My own proposed JSON serialization for MARC [marc-hash](http://robotlibrarian.billdueber.com/2010/02/new-interest-in-marc-hash-json/) should be considered dead: the commuity has all gotten behind MARC-in-JSON.
