---
title: "Indexing data into Solr via JRuby (with threads!)"
date: 2010-02-16
---

[Note: in this post I'm just going to focus on the "get stuff into Solr" part. My normal focus -- MARC data -- will
make an appearance in the next post when I talk about using this in addition to / instead of solrmarc.]


### Working with Solr

I love me the Solr. I love everything about it except that the best way to interact with it is via Java. I don't so much love me the java.

So...taking Erik Hatcher's lead and advice, as I will do whenever he offers either, I wrote some code to work within JRuby to deal with Solr. 

### Getting the code

I've added the gems to gemcutter, if you want to play along at home:

  * **jruby\_producer\_consumer** ([github](http://github.com/billdueber/jruby_producer_consumer), [rdoc.info](http://rdoc.info/projects/billdueber/jruby_producer_consumer)) Ruby syntax for threaded operations under jruby
  * **jruby\_streaming\_update\_solr\_server** ([github](http://github.com/billdueber/jruby_streaming_update_solr_server), [rdoc.info](http://rdoc.info/projects/billdueber/jruby_streaming_update_solr_server)) Ruby syntax on top of the Java class of the same name
  * **marc4j4r** ([github](http://github.com/billdueber/marc4j4r), [rdoc.info](http://rdoc.info/projects/billdueber/marc4j4r)) Ruby syntax on top of the marc4j java library.

*WARNING*: None of these gems have a 1.0 version tag on them, and that means that the API may change a titch in 
  the future. Also, the fact that they're released as gems means that it's easy to release gems, not that I'm not
  an idiot.

### The basics: Using SolrInputDocument and StreamingUpdateSolrServer

OK, with the disclaimer out of the way, let's look at some code.


~~~ruby

  require 'rubygems'
  require 'jruby_streaming_update_solr_server'

  solrurl = 'http://your.solr.server:port/solr'
  sussqueuesize = 24 # how many items to buffer on their way to solr
  sussthreads = 1   # how many threads to use to send stuff to solr
  
  suss = StreamingUpdateSolrServer.new(solrurl,sussqueuesize,sussthreads)
  
  # Let's add a simple document via a hash: A title, three authors, and a year
  
  h = {
    :title => "Never been deader",
    :author => ['Bill', 'Mike', 'Molly'],
    :year => 2003
  }
  suss << h
  suss.commit
  
  # YEA! You just added a document to solr and committed it. 
  # Have a cookie!
  
  # We can also use a document object to do the same thing
  
  doc = SolrInputDocument.new
  # Add the title
  doc << ['title', 'Never been deader']
  
  # Add the first author
  doc << [:author, 'Bill']
  
  # Add more. Re-used keys mean you're adding additional values
  # Note values can be scalars or arrays
  
  doc << [:author, ['Mike', 'Molly']]
  
  # Add the wrong year using [] syntax
  doc[:year] = 2001
  
  # Oops! fix it. []= overwrites existing value(s)
  
  doc[:year] = 2003
  
  # Finally, we can merge a hash (or anything else that responds to 
  # 'each_pair' with key-value pairs) into an existing doc
  
  doc.merge! {'author' => 'Ringo Starrre', 'publisher'=>'Vainity Books'}
  
  # Add it
  
  suss << doc

  # Commit and optimize if you'd like

  suss.commit
  suss.optimize # if you want
  

~~~

Nothing really fancy in there -- just a few things worth noting:

  * An suss object will take a hash (again, anything that responds to `#each_pair`) or a SolrInputDoc
  * You can use either strings or symbols to represent Solr field names
  * Values can be either a single value, or an array of multiple values

And there are three ways to get data into a doc:

  * Via `<< [field, value(s)]` (additive)
  * Via `doc.merge! hash` (additive)
  * Via `doc[field] = value` (replaces)

### Adding Threads

I also went down the garden path of threading things. There are an awful lot
of operations that are not threadsafe (e.g., reading a line from a file) but
once you've got a bunch of records to worth with, turning them into Solr
documents is usually thread-safe.

My model is that there's a producer (usually the method `#each`) from an 
underlying data object. A thread takes whatever that method yields
and sticks the values into a java 
BlockingQueue awaiting consumption. You then use `ProdcuerConsumer#threaded_each`
(or `ProducerConsumer#threaded_each_with_index`) to pull items out of the queue and do something useful with them.

I extracted stuff into a library (jruby\_producer\_consumer) for your viewing pleasure.

**CONFUSION ALERT**: It's perhaps unfortunate that the object you send to `ProducerConsumer.new(obj)` must implement `#each` and that the ProducerConsumer method `#threaded_each` calls that underlying `#each`...well
there's a lot of `#each`'s floating around. Keep them straight.


So...let's look at some code to work with consumer threads.


~~~ruby

  # Start off the same as before
  require 'rubygems'
  require 'jruby_streaming_update_solr_server'
  require 'jruby_producer_consumer'
  require 'marc4j4r'
  
  solrurl = 'http://your.solr.server:port/solr'
  sussqueuesize = 24 # how many items to buffer on their way to solr
  sussthreads = 2   # how many threads to use to send stuff to solr
  
  suss = StreamingUpdateSolrServer.new(solrurl,sussqueuesize,sussthreads)
  
  # I'll go ahead and use a MARC file as my example, but won't talk about the
  # MARC parts of it. All you need to know is that the reader object
  # implements #each
  
  reader = MARC4J4R.reader('test.xml', :marcxml)
  
  # Get a producer/consumer object with the reader at its base, using
  # the default method #each to get stuff out of it, and with the assumption
  # that we only need to keep the default 5 items in memory at a time to 
  # keep up with consumption
  
  pc = ProducerConsumer.new(reader)
  
  # Get three threads to actually consume the things, turn them into solr 
  # documents, and send them to solr (potentially out of order)
  
  numconsumerthreads = 3
  pc.threaded_each(numconsumerthreads).each do |r|
    suss << turn_marc_record_into_a_hash_or_solrdoc(r)
  end
  suss.commit
  

~~~

Again, not a lot happening here. 

  * The "producer" is always one thread, because so little is thread-safe at the 'each' level. In this case, there's a single thread pulling data out of the file and turning it into MARC records, which are added to the internal BlockingQueue. I buffer 5 of these at a pop (the default) so the consumer threads don't starve. I presume that producing items is cheaper than consuming them, or else this library won't help you much. 
  * `ProducerConsumer#threaded_each` calls the `#each` method of the underlying object. You can substitute anything that yields, though, as in this example where I call `#each_line` instead of the default `#each`
  

~~~ruby

  queuesize = 5
  pc = ProducerConsumer.new(File.new('myfile.txt'), queuesize, :each_line)

~~~

  * Keep track of your threads. In this last example, there is one thread getting MARC records and putting them into the PC buffer (no way to change that), three threads consuming those records and sticking them into the `suss` object, and another two pulling stuff *out* of the `suss` object and sending things to Sorl. And, of course, there's other stuff running on the computer, too. Experiment and figure out what works best for your hardware.
  * See the docs for how to mess with what goes into a ProducerConsumer object. It's entirely possible to use, say, `#each_slice`. There's also a convenience method `#threaded_each_with_index`, but it does *not* call the underlying `#each_with_index`, it produces its own index as things are read. 
  
### Feedback not only welcome but necessary!

I've done a lot of messing around with Ruby in the last 10 days or so, but I'm still basically converting from Perl in my head. Any comments, bugs reports, or whatnot are definitely welcome!
