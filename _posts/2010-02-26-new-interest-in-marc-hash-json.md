---
title: "New interest in MARC-HASH / JSON"
date: 2010-02-26
---

<blockquote>EDIT: This is historical -- the recommended serialization for marc in json is now Ross Singer's <a href="http://dilettantes.code4lib.org/blog/2010/09/a-proposal-to-serialize-marc-in-json/">marc-in-json</a>. The marc-in-json serialization has implementations in the core marc libraries for <a href="https://rubygems.org/gems/marc">Ruby</a> and <a href="http://pear.php.net/package/File_MARC/">PHP</a>, and add-ons for <a href="http://search.cpan.org/~bbaxter/MARC-Utils-MARC2MARC_in_JSON/">Perl</a> and <a href="https://github.com/billdueber/marc4j_extra_reader_writers">Java</a>. C'mon, Python people!</blockquote>
For reasons I'm still not entirely clear on (I wasn't there), the Code4Lib 2010 conference this week inspired renewed interest in a JSON-based format for MARC data.

When I initially looked at MARC-HASH <a href="http://robotlibrarian.billdueber.com/marc-hash-the-saga-continues-now-with-even-less-structure/">almost a year ago</a>, I was mostly looking for something that wasn't such a pain in the butt to work with, something that would marshall into multiple formats easily and would simply and easily round-trip.

Now, though, a lot of us are looking for a MARC format that (a) doesn't suffer from the length limitations of binary MARC, but (b) is less painful (both in code and processing time) than MARC-XML, and it's worth re-visiting.

For at least a few folks, un-marshaling time is a factor, since no matter what you're doing, processing XML is slower than most other options. Much of that expense is due to functionality and data-safety features that just aren't a big win with a brain-dead format like MARC, so it's worth looking at alternatives.
<h2>What is MARC-HASH?</h2>
At some point, we'll want a real spec, but right now it's just this:

~~~javascript
  
  // A record is a four-pair hash, as follows. UTF-8 is mandatory.
  {
    "type" : "marc-hash"
    "version" : [1, 0]
    "leader" : "...leader string ... "
    "fields" : [array, of, fields]
  }

  // A field is an array of either 2 or 4 elements
  [tag, value] // a control field
  [tag, ind1, ind2, [array, of subfields]]

  // A subfield is an array of two elements

  [code, value]
~~~

So, a short example:

~~~json
  {
    "type" : "marc-hash",
    "version" : [1, 0],

    "leader" : "leader string"
    "fields" : [
       ["001", "001 value"]
       ["002", "002 value"]
       ["010", " ", " ",
        [
          ["a", "68009499"]
        ]
      ],
      ["035", " ", " ",
        [
          ["a", "(RLIN)MIUG0000733-B"]
        ],
      ],
      ["035", " ", " ",
        [
          ["a", "(CaOTULAS)159818014"]
        ],
      ],
      ["245", "1", "0",
        [
          ["a", "Capitalism, primitive and modern;"],
          ["b", "some aspects of Tolai economic growth" ],
          ["c", "[by] T. Scarlett Epstein."]
        ]
      ]
    ]
  }
~~~
<h2>How's the speed?</h2>
I think it's important to separate the format marc-hash from the eventual marshaling format -- partly because someone might actually want to use something other than JSON as a final format, but mostly because it forces implementations to separate hash-creation from json-creation and makes it easy to swap in different json libraries when a faster one comes along.

Having said that, in real life people are mostly concerned about JSON. So,
let's look at JSON performance.

The MARC-Binary and MARC-XML files are normal files, as you'd expect. The JSON file is
<a href="http://trephine.org/t/index.php?title=Newline_delimited_JSON">"Newline-Delimited JSON"</a> -- a single JSON record on each line.

The benchmark code looks like this:

~~~ruby
  # Unmarshal
  x.report("MARC Binary") do
    reader = MARC::Reader.new('test.mrc')
    reader.each do |r|
      title = r['245']['a']
    end
  end

  # Marshal
  x.report("MARC Binary") do 
    reader = MARC::Reader.new('test.mrc')
    writer = MARC::Writer.new('benchout.mrc')
    reader.each do |r|
      writer.write(r)
    end
    writer.close
  end
~~~~

Under MRI, I used the nokogiri XML parser and the yajl JSON gem. Under JRUby, it was the jstax XML parser and the json-jruby JSON gem.

The test file is a set of 18,831 records I've been using for all my benchmarking of late. It's nothing special; just a nice size.

<h3>Marshalling Speed (read from binary marc, dump to given format)</h3>

Times are in seconds on my Macbook laptop, using ruby-marc.

<table class="grid">
<tbody>
<tr>
<th>Format</th>
<th>Ruby 1.87</th>
<th>Ruby 1.9</th>
<th>JRuby 1.4</th>
<th>Jruby 1.4 --1.9</th>
</tr>
<tr>
<td>XML</td>
<td>393</td>
<td>443</td>
<td>188</td>
<td>356</td>
</tr>
<tr>
<td>MARC Binary</td>
<td>36</td>
<td>23</td>
<td>23</td>
<td>25</td>
</tr>
<tr>
<td>JSON/ NDJ</td>
<td>31</td>
<td>19</td>
<td>25</td>
<td>ERROR</td>
</tr>
</tbody>
</table>

<h3>Unmarshalling speed (from pre-created file)</h3>

Again, times are in seconds

<table class="grid">
<tbody>
<tr>
<th>Format</th>
<th>Ruby 1.87</th>
<th>Ruby 1.9</th>
<th>JRuby 1.4</th>
<th>Jruby 1.4 --1.9</th>
</tr>
<tr>
<td>XML</td>
<td>113</td>
<td>89</td>
<td>75</td>
<td>89</td>
</tr>
<tr>
<td>MARC Binary</td>
<td>29</td>
<td>16</td>
<td>16</td>
<td>19</td>
</tr>
<tr>
<td>JSON/ NDJ</td>
<td>17</td>
<td>9</td>
<td>13</td>
<td>16</td>
</tr>
</tbody>
</table>

<h2>And so...</h2>

I'm not sure what else to say. The format is totally brain-dead. It round-trips. It's fast enough. It has no length limitations. We define it to have to be UTF-8. The NDJ format is easy to understand and allows streaming of large document collections.

If folks are interested in implementing this across other libraries, that'd be great. Any thoughts?
