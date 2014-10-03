---
title: "Why programmers hate free text in MARC records"
date: 2011-04-11
tags: "bad data"
layout: post

---

One of the frustrating things about dealing with MARC (nee AACR2) data is how much nonsense is stored in free text when a unique identifier in a well-defined place would have done a much better job.

A lot of people seem to not understand why.

This post, then, is for all the catalogers out there who constantly answer my questions with, "Well, it depends" and don't understand why that's a problem.

### Description vs Findability

I'm surprised -- and a little dismayed -- by how often I talk to people in the library world who don't understand the difference between _description_ and _findability_. AACR2 is clearly designed for _description_; once you've found a record, it does a pretty good job telling a human being what she's looking at. With respect to a person who's already got a copy of the record in her (virtual) hand, strings of text and reasonable abbreviations are...well, often good enough, let's say.

But much of AACR2 is a giant mountain of fail when it comes to supporting _findability_ -- the ability for a machine to slice and dice the data in ways that can be mapped onto searches and transformations. What those of us on the business end of the computer need are _well-defined values_ stuck into _well-defined places_ that represent _well-defined relationships_.

Free text stuck on the end of a field fails all three of those criteria.

### Machine Reasoning vs. Machine Parsing

When many people look at something like RDF, their first reaction is, "[Great Googally Moogally](http://www.youtube.com/watch?v=ffN9jcVcH_o)!  Just tell me the language! I don't want to follow a chain of reasoning that's seventeen steps long just to figure out the damn thing is in English!!!"

Of course you don't. And you don't have to. Someone -- hopefully someone smarter than me -- needs to write a program to do it. And we can.

Following all that logic -- deriving relationships, figuring out eventual values, determining how to convert between various forms -- is what I'll call (for simplicity's sake) _machine reasoning_. And machine reasoning -- for the purposes of this discussion, anyway -- is a **solved problem**. I'm not saying it's not hard, and I'm not saying it might not take gobs of hardware resources. But we, the collective of humanity, know how to do it.

On the other hand, _machine parsing_ -- looking at all that free text that is sprinkled throughout our records and trying to turn it into something that is susceptible to machine reasoning -- is vehemently _not_ a solved problem. Even if you ignore all the misspellings, we're still stuck with one-off abbreviations, lack of ordering, gobs of "local practice," and iffy punctuation.

And, come to think of it, you can't ignore the misspellings, either.

The point is this: **good data trumps everything else**. If there's good, solid, well-defined data in computable places, we can (given some time) do damn near anything with it. If there's human-entered, free-text, parenthetical-remark-type data, we're pretty much stuck.

### Examples?

[Jonathan Rochkind](http://bibwild.wordpress.com/) just did a [great post](http://bibwild.wordpress.com/2011/04/04/broad-categories-from-class-numbers/) looking at LC call numbers, and how, well, they might be in a few different places, and may or may not be valid LC call numbers, and so on and on and on and on.

And my next post (hopefully tomorrow) will be an analysis of the first freetext in MARC I ever tried to deal with -- the parenthetical remarks in the 020 (ISBN) field. If that doesn't keep you up all night, well, I don't know what will.
