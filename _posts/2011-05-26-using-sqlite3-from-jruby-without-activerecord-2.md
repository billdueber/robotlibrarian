---
title: "Using SQLite3 from JRuby without ActiveRecord"
date: 2011-05-26
tags: "jruby"
layout: post

---

I spent way too long asking my friend, The Internet, how to get a normal DBI connection to SQLIte3 using JRuby. Apparently, everyone except me is [using ActiveRecord and/or Rails](http://jruby-extras.rubyforge.org/activerecord-jdbc-adapter/) and doesn't *want* to just connect to the database.

But I do. Here's how.

First, get the gems:


~~~bash

  gem install dbi
  gem install dbd-jdbc
  gem install jdbc-sqlite3

~~~

Then you're ready to load it up into DBI.


~~~ruby

require 'rubygems' # if you're using 1.8 still
require 'java'
require 'dbi'
require 'dbd/jdbc'
require 'jdbc/sqlite3'

databasefile = 'test.db'
dbh = DBI.connect(
  "DBI:jdbc:sqlite:#{databasefile}",  # connection string
  '',                                 # no username for sqlite3
  '',                                 # no password for sqlite3
  'driver' => 'org.sqlite.JDBC')      # need to set the driver

# That's it. Everything below here is stock DBI

dbh.do "create table squares (i integer, isquared integer)"

ins = dbh.prepare("insert into squares values (?, ?)")
(1..20).each do |i|
  ins.execute(i, i*i)
end


~~~
