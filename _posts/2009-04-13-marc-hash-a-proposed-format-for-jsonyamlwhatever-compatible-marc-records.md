---
title: "MARC-Hash: a proposed format for JSON/YAML/Whatever-compatible MARC records "
date: 2009-04-13
---

In my first shot at MARC-in-JSON, which I appropriately (and prematurely) named <a href="http://code.google.com/p/marc-json/">MARC-JSON</a>, I made a point of losing round-tripability (to and from MARC) in order to end up with a nice, easy-to-work-with data structure based mostly on hashes. "Who really cares what order the subfields come in?" I asked myself.

Well, of course, it turns out some people do. Some even care about the order of the tags. "Only in the 500s...usually" I was told today. All my lovely dreams of using easy-to-access hashes up in so much smoke.

So...I'm suggesting we try something a little simpler. Something so brain-dead, in fact, that I'm loathe to put it down because it's pretty much the obvious way to do it. To wit:


~~~javascript

{
  "type" : "marc-hash",
  "version" : [1, 0],
  
  "leader" : "leader string"
  "control" : [
     ["001", ["all", "001", "values"]],
     ["002", ["all", "002", "values"]],
  ],
  "data" : [
    ["010", " ", " ", 
      [
        ["a", "68009499"]
      ]
    ],
    ["035", " ", " ",
      [
        ["a", "(RLIN)MIUG0000733-B"]
      ],
    ]
    ["035", " ", " ",
      [
        ["a", "(CaOTULAS)159818014"]
      ],
    ]
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

Stupid MARC allows all the stupid fields to stupid repeat and be out of stupid order and such, so it's just a lot of arrays. Easily round-tripable. 

Why bother? Excellent question, and one that's a little harder to answer now that the data structure requires so much looping to find anything (the first time, anyway). I guess it's still a lot easier than working with raw MARC (or, I would claim, MARC-XML), requires no special libraries in any language that supports strings, hashes, and arrays, and can be manipulated with basic language constructs.

A few things worth noting about the assumptions in my mind:

  * By definition, it's always UTF-8. The leader should be changed to note this on the sending end, but it's not required.
  * We include both a type "marc-hash", and a version with major/minor numbers.
  * Everything is a string.
  * Alpha characters in indicators/tags are all lowercased.
  * A control field is a duple: tag and array of values.
  * A data field has four values:
      * The tag
      * Indicator one
      * Indicator two
      * An array of duples: subfield and its value


##A simple transformation to make it a little more queryable

Let's say you don't give a damn about tags that appear out of order, because that's just a crime against nature, anyway. And you really don't care what order the subtags appear in most of the time, 'cause really, who does?

A simple run-through (psuedocode ahead):


~~~perl

  my marchash = getTheMarcHash();
  my kindamarc;
  kindamarc{leader} = marchash{leader};
  
  # Map the control fields by tag => array-of-values
  foreach cfield (marchash{control}) {
    kindamarc{control}{cfield[0] ||= []};
    kindamarc{control}{cfield[0]}.push(cfield[1]);
  }
  
  foreach d (marchash{data}) {
    (tag, ind1, ind1) = (d[0], d[1], d[2]);
    
    # build up a hash based on subfields for this tag
    newd = {};
    foreach subfield (d[3]) {
      (stag, sval) = subfield;
      newd{stag} = sval;
    }
    
    # Store the subfield hash in a few places so it's easy to find.
    foreach i1 ('*', ind1) {
      foreach i2 ('*', ind2) {
        kindamarc{data}{tag}{i1}{i2} ||= [];
        kindamarc{data}{tag}{i1}{i2}.push(newd);
      }
    }
  }

~~~~


Control fields are stored as arrays of values associated with the tag. Data fields are built up as a hash of subfield to array-of-values pairs, and then stored both based on the indicator given and the wildcard indicator '*'.

Basically, this will allow things like this:

~~~perl
  $leader = $kindamarc{leader};
  $first001 = $kindamarc{control}{"001"}[0];
  
  # Find 856s where indicator 2 is '1'
  
  @mystuff = $kindamarc{data}{856}{'*'}{1};  
~~~

It's easy to see how we could store the index from the original array to make it easy to find the original order, too. 

For many, I'm sure, the prospect of dealing with something like this is more daunting than just learning to use MARC-XML or using existing libraries to deal with straight MARC. But there seems to be a set of folks out there for whom this might be useful, so I'm throwing it out there.
