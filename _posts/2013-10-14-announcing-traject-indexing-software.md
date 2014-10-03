---
title: "Announcing \"traject\" indexing software"
date: 2013-10-14
layout: post

---

[Over the next few days I'll be writing a series of posts that highlight a new indexing solution by Jonathan Rochkind and myself called `traject` that we're using to index MARC data into Solr. This is the introduction.]

Wow. Six months since I posted here. What have I been doing?

Well, mostly parenting, but in the last few weeks I was lucky enough to get on board with a project started by [Jonathan Rochkind](http://bibwild.wordpress.com/) for a new JRuby-based tool optimized for indexing MARC data into solr. You know, kinda like solrmarc, but JRuby.


### What's it look like?

I encourage you to take a look at a [little sample setup](https://github.com/traject-project/traject_sample) I put together for instructional purposes. It's based on the HathiTrust catalog indexing scheme and shows off about 85% of what `traject` can do. Clone it and go through the README and the two indexing files to get a taste of how things are put together.

Real quickly, though, here's a sample configuration file to pull out the ID, title, and authors (if any) out of a file of MARC records and send them to a file as JSON object, one record per line (i.e., newline-delimited JSON)



~~~ruby

# we'll pretend this file is called 'sample.rb'
require 'traject'
require 'traject/marc_reader'
require 'traject/json_writer'


# It's just ruby, so I can have comments!
# Here we set up which reader/writer to use and so on
settings do
  provide "reader_class_name", "Traject::MarcReader"
  provide "writer_class_name", "Traject::JsonWriter"
  provide "output_file", "basics.ndj"
  provide 'processing_thread_pool', 3
end


# It's *still* just ruby, so I can declare a variable!
idfield = '001'

# ...and then use it to find the ID
to_field "id", extract_marc(idfield, :first => true)

# Now the other data
to_field "title", extract_marc('245')
to_field "author", extract_marc('100abcd:110abcd:111abc')


# You'd run this as:
#    traject -c sample.rb myfile.mrc


~~~

That's simplistic, of course, but it should drive home the point that we strove to make sure traject _makes the easy stuff easy_.  For a more complex example, look at [the heavily-annotated index.rb file](https://github.com/traject-project/traject_sample/blob/master/index.rb) in the sample project.

### Why use (or move to) traject?

First off, you can and should look at [the annoucement](http://bibwild.wordpress.com/2013/10/14/traject-marc-solr-indexer-release/) and/or [the README](https://github.com/traject-project/traject/) for a longer answer, but I'll tell you why _I_ use `traject` in one word:

Flexibility.

After a year or so of struggling with [solrmarc](https://code.google.com/p/solrmarc/) (often due to my lack of Java-fu), and then even more years after that using my own, home-grown [marc2solr](https://github.com/billdueber/marc2solr), the things I most wanted were the ability to decouple the various components from each other, rely on code instead of configuration, and basically just know that I can up the complexity of my code without paying an enormous price.

I'm fast wtih Ruby. And the architecture of `traject` allows me to easily build and test my transformations in isolation, with tools I'm good with, with debugging output that's easy to read or process by machine or inspection.

### What does it have out of the box?

One advantage `traject` has that my previous system didn't is, well, years of struggling with my previous system. I've learned a lot about what I need, what needs to be easy, and how I want to think about indexing.

The nature of `traject` is that "a reader" sends "a record" to "an indexer" which produces a key=>value hash and sends _that_ to "a writer." Obviously, this is a pretty abstract setup; it's not hard to see how it could be used for all sorts of transformations (e.g., I'm already thinking about a simple gem that would provide macros to index CSV or tab-delmited files into Solr. Or maybe going to/from a database).

But Jonathan and I are, mostly, stuck dealing with MARC data and Solr. So here's what we get:

**Readers**: MARC readers for MARC21 binary and MARC-XML based on both ruby-marc and marc4j (the latter allowing you to deal with encoding transformations and the like). An NDJ reader (for one marc-in-json structure per line in a file -- that's what we use in for the HathiTrust). And we've already got a couple gems for people with other needs: [traject_alephsequential_reader](https://github.com/traject-project/traject_alephsequential_reader) for those that need to deal with AlephSequential, and Jonathan's new [horizon reader](https://github.com/jrochkind/traject_horizon) for efficiently pulling records right out of your Horizon ILS, if you happen to run one.

**Transforming Macros**: A traject indexing step is just a well-formed ruby block (or lambda), which makes writing macros ridiculously easy. Traject ships with most of what you'd commonly need to deal with MARC: extracting data based on tag/subfield/indicators (or substring of a fixed field), dealing with non-filing characters, automatically dealing with 880 linked fields. Mucking with publication dates. Dealing with languages, formats, etc. And, of course, doing it all with multiple threads, because who wants to see all those lovely cores go to waste?

**Writers**: Of course, you can write to solr, using the excellent `solrj` java library. And you can do it in multiple threads, to keep things fast. But there's also the `DebugWriter` to spit stuff out in a human-readable format, and the `JsonWriter` mentioned above to spit stuff out in a _machine_-readable format. And building your own writer is literally just a couple methods.

### How do I get a taste?

Like I said, clone and play with [the sample project](https://github.com/traject-project/traject_sample). And ask me questions, either here or via email. After years of being the only person running my indexing software, I'm anxious to try to build up a community around `traject`.
