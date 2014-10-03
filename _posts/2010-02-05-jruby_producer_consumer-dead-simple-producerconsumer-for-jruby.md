---
title: "jruby_producer_consumer dead-simple producer/consumer for JRuby"
date: 2010-02-05
---

Yea! My first gem ever released!

<p style="color: red">[YUCK! It was a disaster in a few ways! Don't look at this! It's hideous! There's a new <a href="http://rdoc.info/projects/billdueber/jruby_producer_consumer">jruby_producer_consumer</a> gem on gemcutter that is slightly different from this in that it works. Ignore the stuff below.]</p>

[In working on a threaded JRuby-based MARC-to-Solr project, I realized that my threading stuff was...ugly. And
I didn't really understand it. So I dug in today and wrote this.]

I've just pushed to [Gemcutter](http://gemcutter.org/) my first gem -- a [JRuby](http://jruby.org/)-only
producer/consumer class that works with anything that provides _#each_ called [jruby_producer_consumer](http://gemcutter.org/gems/jruby_producer_consumer).

It's JRuby-only because it uses (a) A blocking queue implemenation that's native Java, and (b) threading, which isn't 
a huge win under regular Ruby.

There's no testing there because I'm not sure how to test threaded stuff :-(

It is, I hope, easy to use:


~~~ruby

   require 'rubygems'
   require 'jruby_producer_consumer'

   # Create a ProducerConsumer. Arguments are anything that implements #each
   # and the size for the underlying queue. For the former, I'll just use a Range object.

   eachable = 1..10
   queuesize = 3

   pc = ProducerConsumer.new(eachable, queuesize)

   # Just a method to show what happens
   def sample (consumerid, x)
     puts "Consumer #{consumerid}: consuming #{x}"
     sleep 1 # otherwise this'll finsish before I can create multiple consumers
   end

   # Create three consumers. You can pass any number of args to
   # #consumer, and must pass a block whose arguments are the
   # object returned by eachable#each and those args back.

   ['A', 'B', 'C'].each do |consumerid|
     pc.consumer(consumerid) do |x, consumerid|
       sample(consumerid, x)
     end
   end

   # OUTPUT
   # Consumer A: consuming 1
   # Consumer B: consuming 2
   # Consumer C: consuming 3
   # Consumer A: consuming 4
   # Consumer B: consuming 5
   # Consumer C: consuming 6
   # Consumer B: consuming 7
   # Consumer A: consuming 8
   # Consumer C: consuming 9
   # Consumer B: consuming 10


~~~
