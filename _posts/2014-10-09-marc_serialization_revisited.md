---
title: Ruby MARC serialization/deserialization revisited
layout: post
date: 2014-10-09 12:58

---

A few years ago, I [benchmarked](http://robotlibrarian.billdueber.com/2010/09/sizespeed-of-various-marc-serializations-using-ruby-marc/)
various methods of serializing/deserialzing MARC data using the [ruby-marc](http://github.com/ruby-marc/ruby-marc/)
gem. Given that I'm planning on starting fresh with my catalog setup, I thought I'd take a moment
to revisit them.

The biggest changes since that time have been (a) the continued speed improvements in JRuby,
(b) the introduction of the [Oj](https://github.com/ohler55/oj) json parser for MRI ruby, and
(c) wider availability of msgpack code in the wild.

I also wondered what would happen if I tried ruby's `Marshal` serialization; maybe it would be
faster because I wouldn't have to "manually" create a `MARC::Record` object from a hash?

## File sizes

File size isn't as important as it once was, but still matters to some of us working with
ginormous amounts of data:

This is the file size to hold the 18,881 records used for the benchmark.

|-
| Serialization | Size on disk (MB) |  Size vs. marc21 | Gzipped size on disk (MB) | Gzipped size vs marc21 |
|:--------------|------------------:| -------------------------:|--:|--:|
| marc21 | 31 |100% | 9.0 | 100%|
| msgpack | 42 | 135% | 8.2 | 91% |
| json (ndj) | 56 | 180% | 8.1 | 90% |
| marshal | 69 | 223% | 9.4 | 104% |
| marc-xml | 93 | 300% |  9.2 | 102% |

It's interesting, if not super-useful, to note that the file sizes differ by a factor of three
uncompressed, but hardly at all when compressed. I was surprised at how well the binary
formats (msgpack, marshal, and marc21) compressed.

## Serialization / Deserialization time

I took a file of about 19k MARC records and tested the serialization/deserialization time, as follows:

* **marc21** uses MARC::Reader and MARC::Writer from the ruby marc distribution
* **json** Uses `MARC::Record#to_hash` to produce a marc-in-json hash, serializes with the stock JSON library, and
the writes to a file with one record per line (sometimes known as *newline-delimited JSON, or NDJ*). Deserialization
reverses the process
* **json (oj)** is the same, except using the Oj json library under MRI.
* **msgpack** uses the [msgpack](https://github.com/msgpack/msgpack-ruby) or [msgpack-jruby](https://github.com/iconara/msgpack-jruby) gems to
serialize/deserialize msgpack objects to/from a file stream.
* **marshal** uses the core ruby [Marshal](http://www.ruby-doc.org/core-2.1.3/Marshal.html) class to serialize/deserialize to a file stream.

In all cases, deserialization means to pull each record in turn from a file on disk and turn it into a `MARC::Record` object;
serialization means to take a set of pre-created `MARC::Record` objects, serialize them, and push them into a file.

All times are in "real time" seconds as reported by Benchmark, averaged across two runs on my desktop machine:

* **MRI** ruby 2.1.2p95 (2014-05-08 revision 45877) [x86_64-darwin13.0
* **JRuby** jruby 1.7.15 (1.9.3p392) 2014-09-03 82b5cc3 on Java HotSpot(TM) 64-Bit Server VM 1.8.0-b132 +indy +jit [darwin-x86_64]

[The benchmark code](https://gist.github.com/billdueber/e375a35ebabd2de73616) is up on a gist if you'd like
to look at or modify it.

| MRI Ruby
| Method | Deserialize (s)  | Serialize (s) | Total (s) | Total time vs. marc21 |
|--------|------------:|----------:|-------:|-----------------------:|
| marc21 | 11.5 | 6.7 | 18.2 | 100% |
| json   | 9.5  | 5.9 | 15.4 | 85%
| json (oj) | 6.4 | 3.5 | 9.9 | 54% |
| msgpack | 5.8 | 3.4 | 9.2 | 51% |
| marshal | 7.0 | 8.7 | 15.7| 86% |

| JRuby
| Method | Deserialize (s)  | Serialize (s) | Total (s) | Total time vs. marc21 |
|--------|------------:|----------:|-------:|-----------------------:|
| marc21 | 6.9 | 4.2 | 11.1 | 100% |
| json   | 4.1  | 2.5 | 6.6 | 59% |
| json (oj) | N/A | N/A | N/A | N/A |
| msgpack | 4.0 | 3.9 | 7.9 | 71% |
| marshal | 5.3| 8.1 | 13.4| 121% |

## Conclusions

* JRuby is faster than MRI on this these tasks, at least once it's warmed up.
* JSON, with a decent library, is either the fastest (JRuby) or really close (MRI)
* Marshal is slow and big.

I come away with this thinking the same thing I did last time. I'm going to use compressed ndj in files, and (possibly compressed) JSON
over the wire. The speed is great, tool support is outstanding, and having something human-readable is a big bonus.



