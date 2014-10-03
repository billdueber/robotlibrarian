---
title: "Data structures and Serializations"
date: 2010-04-20
---

Jonathan Rochkind, in response to a long (and, IMHO, mostly ridiculous) thread on NGC4Lib, has been exploring the boundaries between a data model and its expression/serialization (
see [here](http://bibwild.wordpress.com/2010/04/19/of-marc-serialization-formats-and-element-schemata/), [here](http://bibwild.wordpress.com/2010/04/19/and-more-on-software-data-formats/), and [here](http://bibwild.wordpress.com/2010/04/20/serialization-vs-metadata-schemavocabulary/)
) and I thought I'd jump in. 

### What this post is *not*

There's a lot to be said about a good domain model for bibliographic data. I'm *so* not the guy to say it. I know there are arguments for and against various aspects of the AACR2 and RDA and FRBR, and I'm unable to go into them. 

What I *am* comfortable saying is this:

> Anyone advocating or dismissing a data model based on the  data structure or serialization most-often associated with that model **is missing the goddamn point**.


### Data serializations

...are boring. They're unimportant at the data modeling stage, and only barely important when thinking about data structures. For any given data structure there are lots of ways you can serialize it. A standard programming-language hash can be represented in a zillion ways, for example: yaml, json, various programming languages, .ini files, etc. Even MARC has two standard serializations (binary and xml) with several more actually in use (Aleph Sequential, for example). 

So, let me repeat again, serializations are boring and not worth talking about until you've got everything else nailed down. Any format you can round-trip your data structure to/from is fine. 

Serializations are measured from "less pain" to "more pain", but all have the exact same *expressiveness*. Data *structures*, on the other hand, do not.


### A hierarchy of data structures

Think about the following data structures:

*  An ordered list
*  key-value pairs
*  A hierarchy (e.g., an XML document)
*  An undirected graph
*  A directed graph
*  A labeled, directed multigraph (e.g., a set of RDF Triples)


You don't have to think very hard to see that any of these can be viewed as a restricted version of the data structures above it. An ordered list (array) is just a set of key-value pairs where the keys represent each item's sequence. A set of key-value pairs is a very, very flat hierarchy. A hierarchy is an undirected graph without cycles. An undirected graph is a directed graph where you're careful to make links both ways. And a directed graph can easily be represented as a set of RDF triples (where you may, for example, only have one label for your relationships: "links to"). 

[**Note that I didn't say any of these would be efficient implementations**!]

The reverse is not true -- or, at least, not without an incredible amount of "out of band" information in another layer somewhere. 

The structures at the end of the list have more *expressiveness*. You can just plain model more things in them (give-or-take the out-of-band stuff, composition, etc) per unit of screwing around. I'm not going to try to model my set of key=value pairs in an array. I could *do* it, but it would take so much of my attention that the data modeling would suffer. 


### Don't handicap yourself

Don't start with the data structure. 

DON'T START WITH THE DATA STRUCTURE!

GET THAT MOTHER-FREAKIN' DATA STRUCTURE OFF MY MOTHER-FREAKIN' PLANE!

Seriously. Don't be stupid. If all you've got is a hammer, everything starts to look like a thumb.

If you start off with a restrictive data structure before you even fully define the domain you're trying to model, you may hose yourself. You may end up making stupid decisions based on the toolchain you're imagining in your head.

Domain modeling is *ridiculously hard* for any domain worth modeling. If you start with a handicap (a restrictive data structure) it's going to be even harder.

No one would think of trying to model bibliographic data using only arrays. That's premature optimization on an epic scale. 

### The appeal of RDF Triples

Even if you ignore all the semantics and rules that make RDF Triples a value-added instance of a labeled, directed multigraph, the appeal (to me, anyway) is that *any semantic model based on RDF Triples has enormous expressive power at its disposal*. 

Does it turn out that after you've fully satisfied the necessary model for the domain, the semantics you need can actually be accomplished with something lower down in the list? Awesome. Go with it. You'll get great implementations with good real-life computing characteristics. A database can often usefully be thought of an implementation of an undirected graph with typed nodes (and, perhaps, some typed links, if you use the column name in the calling table a "type" of sorts, and add some out-of-band knowledge). And lord knows RDBMS's have great performance characteristics.

But don't *start* there. Start with the domain. Model it. Figure out what you need to describe and derive. *Then* pick the most appropriate data structure.

### The nightmare that is MARC

MARC-the-data-structure (not to be confused with a serialization of that data structure, on the one hand,  or with the AACR2 on the other) can incompletely (but usefully, I think) be described as:

*  A set of key-value pairs
*  ...that have a defined order
*  ...where keys can be repeated
*  ...and values are strings
*  ...and keys are a concatenation of tag/ind1/ind2/code

Control fields are especially restricted (ind1, ind2, and code are all 'null'). There's been some bullshit attempts at links (e.g., the 880 fields) but really, this is it. 

It doesn't give us much to work with. It's restricted. And, sadly, so is our thinking. 

### Putting the cart before the horse

As Jonathan (and zillions of others) rightly point out, a huge problem in the library world is that there are generations (plural) of working librarians who, because of years of practice, find it incredibly hard to think about bibliographic data as modeled outside the constraints inherent in the MARC *data structure*. It's a handicap. It's an anchor around our necks. 

MARC-the-data-*model* (nee AACR2) is not inherently bad because it's built on an impoverished data structure. It's bad because it does a shitty job at modeling the bibliographic data space. If we could produce a good model in a crappy data structure like that, well, that'd be awesome because it would indicate that things are simple. 

Things, of course, aren't simple. They're *hard*. 

So, if you want to complain about MARC or RDA or FRBR, figure out what its trying to model and talk about the fidelity of the model with respect to the problem space. But don't conflate data models, data structures, and serializations. 

Oh, and don't say "PIN Number" or "ATM Machine." That drives me crazy, too.
