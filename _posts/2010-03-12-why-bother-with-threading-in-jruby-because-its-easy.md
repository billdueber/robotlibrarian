---
title: "Why bother with threading in jruby? Because it's easy."
date: 2010-03-12
layout: post
slug: why-bother-with-threading-in-jruby-because-its-easy

---

[<strong>Edit 2011-July-1</strong>: I've written a jruby_specific threach that takes advantage of better underlying java libraries called <a href="http://robotlibrarian.billdueber.com/even-better-even-simpler-multithreading-with-jruby/">jruby_threach</a> that is a much better option if you're running jruby]

Lately on the #code4lib IRC channel, several of us have been knocking around different versions (in several programming languages) of programs to read in a ginormous file and do some processing on each line. I noted some speedups related to multi-threading, and someone (maybe *rsinger*?) said, basically, that to bother with threading for a one-off simple program was a waste.

Well, it turns out I've been trying to figure out how to deal with threading in jruby anyway. And I think I have a pretty elegant solution -- a generic "threaded each" I'm calling `threach`.


~~~ruby

  enumerable_object.threach(number_of_threads, :which_iterator) do |i|
    do_something_threadsafe(i)
  end

~~~

### Some examples


~~~ruby

  # You like #each? You'll love...err..probably like #threach
  load 'threach.rb'

  # Process with 2 threads. It assumes you want 'each'
  # as your iterator.
  (1..10).threach(2) {|i| puts i.to_s}  

  # You can also specify the iterator
  File.open('mybigfile') do |f|
    f.threach(2, :each_line) do |line|
      processLine(line)
    end
  end

  # threach does not care what the arity of your block is
  # as long as it matches the iterator you ask for

  ('A'..'Z').threach(3, :each_with_index) do |letter, index|
    puts "#{index}: #{letter}"
  end

  # Or with a hash
  h = {'a' => 1, 'b'=>2, 'c'=>3}
  h.threach(2) do |letter, i|
    puts "#{i}: #{letter}"
  end

~~~

_threach.rb_ adds to the Enumerable module to provide a threaded
version of whatever enumerator you throw at it (`each` by default).

### How does it work?

How about I just put the source here. It's short.


~~~ruby

  require 'thread'
  module Enumerable

    def threach(threads=0, iterator=:each, &blk)
      if threads == 0
        # Just call the iterator itself
        self.send(iterator, &blk)
      else
        bq = SizedQueue.new(threads * 4)
        consumers = []
        threads.times do |i|
          consumers << Thread.new do
            until (a = bq.pop) === :end_of_data
              blk.call(*a)
            end
          end
        end

        # The producer
        count = 0
        self.send(iterator) do |*x|
          bq.push x
          count += 1
        end
        # Now end it
        threads.times do
          bq << :end_of_data
        end
        # Do the join
        consumers.each {|t| t.join}
      end
    end
  end


~~~

That's it. If threads=0, just use the iterator itself. If not:

* Create a SizedQueue. It is thread-safe by definition and acts as the glue between the consumers and the main-thread producer.
* Start a set of consumer threads that basically just pull an item out of the queue and then run the given block on it. Bail when you see the `end_of_data` token. These consumer threads all immediately block because there's nothing in the SizedQueue yet.
* Populate the SizedQueue. When you run out of stuff to add, push on an `end_of_data` token for each consumer thread.
* Call `join` on the threads to keep the main program around when one of them exits.

### Why use it?

Well, if you're using stock ruby -- you probably shouldn't. It'll just slow things down. But if you're using a ruby implementation that has real threads, like JRuby, this will give you relatively painless multi-threading.

You can always do something like:


~~~ruby

  if defined? JRUBY_VERSION
    numthreads = 3
  else
    numthreads = 0
  end

  my_enumerable.threach(numthreads) {|i| ...}

~~~

Note the "relatively" up there. The block you pass still has to be thread-safe, and there are many data structures you'll encounter that are *not* thread-safe. Scalars, arrays, and hashes are, though, under JRuby, and that'll get you pretty far.
