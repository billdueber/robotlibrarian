---
title: "Even better, even simpler multithreading with JRuby"
date: 2011-07-01
tags: "jruby, ruby"
---

[Yes, another post about ruby code; I'll get back to library stuff soon.]

Quite a while ago, I released a little gem called <em><a href="https://rubygems.org/gems/threach">threach</a></em> (for "threaded #each"). It allows you to easily process a block with multiple threads.


~~~ruby

  # Process a CSV file with three threads
  FIle.open('data.csv').threach(3, :each_line) {|line| send_to_db(line)}

~~~

Nice, right?

The problem is that I could never figure out a way to deal with a <code>break</code> or an <code>Exception</code> raised inside the block. The core problem is that once a thread trying to push/pop from a ruby <code>SizedQueue</code> is blocking, there's no way (I could find) to tell it to wake up and see if there's an error from another thread floating around that needs to be addressed.

So, I got into a pattern of running my code with <code>each</code> for a while, debugging, and eventually doing the production run under <code>threach</code>. Which is just dumb. Then I'd try to re-write <code>threach</code> to deal with this stuff using different approach (mutexes, lightweight events), quickly (or not so quickly) fail, give up, and start again.

So...let's not worry MRI for the moment. I run all my big jobs under JRuby these days anyway, and there I can take advantage of Java's blocking queues that have timeouts. When a queue operation times out, I can check to see if there's been a break or an exception thrown in the meantime and behave appropriately.

The result is the gem <code><a href="https://rubygems.org/gems/jruby_threach">jruby_threach</a></code>. It works just like <code>threach</code>, except that, you know, it actually works the way I'd like it to.


~~~ruby

require 'jruby_threach'
FIle.open('data.csv').threach(3, :each_line) {|line| send_to_db(line)}

~~~

Looks familiar, doesn't it.

But you can also break out of the loop.


~~~ruby

myarray.threach(2) do |item|
  break if item_indicates_to_break(item)
  if item == :really_bad_value
    raise RuntimeError.new, "Something's really wrong", nil 
  end
  process_item(item)
end

~~~

Any exceptions that are <code>rescue</code>d within the block are handled internally and don't cause processing to stop. Any that are <em>not</em> handled within the block are noticed by <code>threach</code>, cause the processing to stop, and the re-raised so you can deal with them outside of <code>threach</code>


~~~ruby


reader = SpecializedFileReader.new(filename)

begin
  reader.threach(2) do |item|
    process_item(item)
  end
rescue SpecializedFileReaderError 
  # deal with the fact that the reader failed
rescue Exception
  # deal with the problem processing the item
end


~~~

Dealing with the underlying Java data structures makes life a lot easier. To the point that I added an enhancement -- threading production as well.


~~~ruby

  # Use two threads to read lines from files, and another three threads
  # to process the data that comes out of those files.
  Dir.glob("*.csv").map{|f| File.open(f)}.mthreach(2,3) do |item|
    send_item_to_datbase(item)
  end

~~~

<code>mthreach</code> basically allows you to treat an array of Enumerables as a single logical entity, multithreading both the producer and consumer sides of the operation. There aren't a whole lot of obvious use cases, but it can certainly come in handy.

You can also access the underlying class that aggregates multiple enumerables directly.


~~~ruby

require 'jruby_threach'
me = Threach::MultiEnum.new(
  [enum1, enum2, enum3], # enumerables
  threads,               # How many threads to use to 
  :each_with_index,      # the iterator to call on the enumerables
  size                   # size of the under-the-hood queue
)

# Note that like threach, calling #each against an MultiEnum actually
# calls the iterator you sent in (in this case, #each_with_index)
me.each {|item| process_item(item)}

~~~
