---
title: "ruby-marc with pluggable readers"
date: 2010-03-02
layout: post

---

I've been messing with easier ways of adding parsers to ruby-marc's MARC::Reader object. The idea is that you can do this:


~~~ruby

  require 'marc'
  require 'my_marc_stuff'

  mbreader = MARC::Reader.new('test.mrc') # => Stock marc binary reader
  mbreader = MARC::Reader.new('test.mrc' :readertype=>:marcstrict) # => ditto

  MARC::Reader.register_parser(My::MARC::Parser, :marcstrict)
  mbreader = MARC::Reader.new('test.mrc') # => Uses My::MARC::Parser now

  xmlreader = MARC::Reader.new('test.xml', :readertype=>:marcxml)

  # ...and maybe further on down the road

  asreader = MARC::Reader.new('test.seq', :readertype=>:alephsequential)
  mjreader = MARC::Reader.new('test.json', :readertype=>:marchashjson)

~~~

A parser need only implement `#each` and a module-level method `#decode_from_string`.

Read all about it [on the github page](http://github.com/billdueber/ruby-marc-plugable-readers).
