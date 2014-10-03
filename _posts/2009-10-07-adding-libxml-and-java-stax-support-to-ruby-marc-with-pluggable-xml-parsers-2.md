---
title: "Adding LibXML and Java STAX support to ruby-marc with pluggable XML parsers"
date: 2009-10-07
layout: post

---

JRuby is my ruby platform of choice, mostly because I think its deployment options in my work environment are simpler (perhaps technically and certainly politically), but also because I have high, high hopes to use lots of super-optimized native java libraries. The [CPAN](http://cpan.perl.org/) is what keeps me tethered to Perl, and whether or not you like Java-the-language, boy, are there a lot of high-quality libraries out there.

Since I've been messing around with MARC-XML parsing of late, and since Ross Singer added [pluggable xml-parser awesomeness](http://groups.google.com/group/blacklight-development/browse_thread/thread/b6f7e064c0cf0bbd/9cd076ca7eeb1293?hl=en&amp;lnk=gst&amp;q=a+few+rubymarc+announcements#9cd076ca7eeb1293) to the ruby-marc project, I thought I'd see what I could do with native Java methods when parsing MARC-XML.

And just for kicks, I threw in the old code that I wrote before that uses LibXML.


### Why do this at all?

Because...er...there's an obvious work-situation where I need to squeeze every last drop of speed out of...ruby...which we don't use...er...

Because. Because I wanted to screw around with the technologies. Because I wanted to learn about calling java native stuff. Because I already wrote the libxml stuff. Because it feels silly to run on the JVM and not use JVM-native code to deal with XML, given that standard java projects make it seem like Java is a giant XML processor with a language wrapped around it.


### What exactly did I do?

For the [LibXML](http://libxml.rubyforge.org/) stuff, I copied my own code. For the java [stax](http://en.wikipedia.org/wiki/StAX) (javax.xml.stream.XMLInputFactory.StreamReader) parser, I stole just about everything from Ross's nokogiri code and put it into its own module, and then slimmed down the nokogiri module and the stax module to only include their differences.


[The patch](http://rubyforge.org/tracker/index.php?func=detail&aid=27253&group_id=964&atid=3783) is at the [ruby-marc rubyforge site](http://rubyforge.org/projects/marc/) if you want to play along at home.


Other than using the stax or libxml parser, everything else is the same -- MARC::Record objects and their components are created exactly as they are with the other parsers. It might be "fun" (for some twisted definition of "fun") to wrap the MARC::Record interface around [marc4j](http://marc4j.tigris.org/) at some point, but right now all that's changed is the parsing.

### Do they work?

Yes. Thanks for asking. At least all the tests pass when I type 'rake'.


### How fast is it?

As always, the numbers are iffy. These were done on my desktop, with other stuff going on. I didn't bother to benchmark `rexml` because we know how slow *that* is.

The test file is a nightly dump intended to go into our VuFind install. It was born as binary marc, and changed to marc-xml using `yaz-marcdump`, which is so fast that I thought maybe something had gone wrong. Holy cow, is `yaz-marcdump` fast.

The resulting XML is 219MB and contains 46,242 records.

The test was to open it up, loop through the records, and pull the 245 out of each. Each segment looks something like this:


~~~ruby

  reader = MARC::XMLReader.new(filename, :parser=>'jstax')
  reader.each do |record|
    title = record[245]
  end

~~~

Times are in seconds. I ran each one five times, with the exception of `jrexml`, during which I got bored.  And the perl code, for which I just wanted to get a ballpark to compare.


~~~

MRI 1.8.7
    libxml     104    (103, 103, 106, 104, 103)
    nokogiri   301    (304, 300, 301, 301, 300)

JRuby trunk
    jrexml     547    (539, 554)
    jstax      203    (201, 208, 201, 201, 204 )

Perl 5.10 w/MARC::File:XML
    perl       340    (340)

~~~~



#### So...faster, right?

Pretty much, yeah.

Under (MRI) ruby, Ross found that nokogiri was 3.5x faster than rexml, and my noodling-around at home showed the same speedup. Using that as a baseline, we get the following speed comparison table using the libxml time normalized to 1.00.

In case that wasn't clear: lower numbers are better.

    libxml:   1.00
    jstax:    1.95
    nokogiri: 2.89
    jrexml:   5.16
    rexml:    10.11 (estimated; 3.5x nokogiri's speed)


### What does it all mean?

It means that adding pluggable parsers was freakin' brilliant.

It means that a guy like me -- with no real expertise in *any* of the applicable technologies -- can do a passable job at integrating a java library into JRuby.

And it means that if I (a) can get folks around here to use Ruby, and (b) can get them to use MARC-XML instead of binary MARC (which we can't use anyway because of the record-length limitations), I can be sure that any bottlenecks aren't going to be the result of those choices.
