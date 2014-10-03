---
title: "Size/speed of various MARC serializations using ruby-marc"
date: 2010-09-29
layout: post

---

[Ross Singer](http://dilettantes.code4lib.org/) recently updated [ruby-marc](http://marc.rubyforge.org/) to include a `#to_hash` method that creates a data structure that is (a) round-trippable without any data loss, and (b) amenable to serializing to JSON. He's calling it *[marc-in-json](http://dilettantes.code4lib.org/blog/2010/09/a-proposal-to-serialize-marc-in-json/)* (even though the serialization is up to the programmer, it's expected most of us will use JSON), and I think it's the way to go in terms of JSON-able MARC data.

I wanted to take a quick look at the space/speed tradeoffs of using various means to serialize MARC records in the marc-in-json format compared to using binary MARC-21.

### Why bother?

Binary MARC-21 is "broken" in that a lot of us have records that are so long (more than 99,999 bytes) it's impossible to create a valid marc binary record. The standard alternative, MARC-XML, has huge filesizes (roughly 3 times as large) and runs a lot more slowly in every benchmark I've ever run. For ruby-marc, the penalty for using XML is further exaggerated because the serializer is based on REXML and is super-slow.

There have been a few proposals for a MARC data structure that can easily be serialized to JSON (I had [my own](http://robotlibrarian.billdueber.com/new-interest-in-marc-hash-JSON/), in fact), but the stuff Ross has done with marc-in-json is preferable in being (a) not a ton bigger in terms of file size, and (b) much easier to query from a [NoSQL](http://nosql-database.org/) database using something like [JSONPath](http://goessner.net/articles/JsonPath/) or [JSONQuery](http://www.sitepen.com/blog/2008/07/16/JSONquery-data-querying-beyond-JSONpath/).

### What I'm testing

For this test, I used:

* **marc21 binary** This is the stock serialization / deserialization provided by ruby-marc.
* **YAJL for JSON** [YAJL](http://lloyd.github.com/yajl/) is a very fast C-based JSON library. Here we're using the Ruby bindings and calling `Yajl::Encoder.encode(r.to_hash)` to serialize and `MARC::Record.new_from_hash(Yajl::Parser.parse(JSON))` to deserialize.
* **Msgpack** The [Msgpack project](http://msgpack.org/) is explicitly designed to be "binary JSON" -- smaller, faster, etc -- at the expense of human readability/editabilty . Again, this used the ruby bindings.

### The benchmark and its results

I'm interested in how long it takes to serialize and deserialize *a single record*. My primary use-case is sticking a single record into Solr, and then pulling the string representation of that record out and turning it back into MARC.

It's entirely possible that trying to deal with a whole set of MARC records -- as a JSON array of marc-in-json objects, or as a set of newline-delimited JSON (or perhaps [LDJSON](https://en.wikipedia.org/wiki/Line_Delimited_JSON) or Msgpack objects -- would yield different results. The former is especially interesting, since to parse a large JSON array one needs to use a streaming parser, which will almost certainly have a different profile in both processing and memory use.

The ambitious can see the [full source code of the benchmark](https://gist.github.com/billdueber/601397).

Note that the following represent only the performance of ruby-marc and the particular serializers used. Other platforms or other libraries will certainly give different results!

Total of 18880 records run 20 times (377,600 serialize/deserialize cycles per method) on my Mac OSX desktop; comparisons are to MARC21-Binary.




~~~

SERIALIZING
MARC Binary 357.02 s (100%)
YAJL 312.65 s ( 88%)
Msgpack 266.26 s ( 75%)

DESERIALIZING
MARC Binary 648.91 s (100%)
YAJL 507.64 s ( 78%)
Msgpack 459.73 s ( 71%)

SERIALIZE + DESERIALIZE
MARC Binary 1005.93 s (100%)
YAJL 820.29 s ( 82%)
Msgpack 725.99 s ( 72%)

SIZE
MARC Binary 31.15 MBytes (100%)
Msgpack 42.00 MBytes (135%)
JSON 55.99 MBytes (180%)
XML 93.42 MBytes (300%)

~~~~

## Analysis, such as it is

Obviously, there are size/speed tradeoffs. Nothing is as small as binary MARC21, but both YAJL and Msgpack are faster -- significantly so for deserialization, which happens to be where I want the speed for my uses.

At 80% larger, the JSON serialization is quite a big bigger, but it's a hell of a lot smaller than MARC-XML and suffers none of the limitations of binary MARC.

For a closed system (i.e., you're not worried about anyone else being able to read your data) such as a Blacklight installation, I'd be tempted to move to using JSON sooner rather than later.
