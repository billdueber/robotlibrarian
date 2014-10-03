---
title: "Stupid catalog tricks: Subject Headings and the Long Tail"
date: 2010-04-13
---

Library of Congress Subject Headings (LCSH) in particular.

I've always been down on LCSH because I don't understand them. They kinda look like a hierarchy, but they're not really. Things get modifiers. Geography is inline and ...weird. 

And, of course, in our faceting catalog when you click on a linked LCSH to do an automatic search, you often get nothing but the record you started from. Which is super-annoying. 

So, just for kicks, I ran some numbers. 

### The process

I extracted all the field 650, indicator2="0" from our catalog, threw away the subfield 6's, and threw away any trailing punctuation in any of the subfields.  I called the concatenation of what was left a unique LCSH. 

Then I printed them out and put them all onto index cards, using tick-marks to indicate...

No, of course not. I used `sort`, `uniq -c`, and `wc -l`. Here's what I found.

### Counts of LCSH

...in round numbers. 

In our catalog, there are:

*  8.50M subject headings (using the definition above)
*  1.87M unique subject headings
*  ...66% of which (1.23M) appear exactly once

We only have to go out to 30K subjects to account for half of all subject entries. The top 1000 most-used subjects account for 14.5% of all 8.5M subject entries. 

The top ten subjects by count are:
 
*  6029 $$aSermons, American
*  6131 $$aPhilosophy
*  7224 $$aFeature films
*  7591 $$aPiano music
*  7968 $$aSocialism
*  8796 $$aEconomics
*  9185 $$aCommunism
* 12440 $$aSermons, English$$y17th century
* 13539 $$aBills, Private$$zUnited States
* 58823 $$aEconomics$$xHistory$$vSources

### From a record's point of view

Our catalog has:

*  7M records
*  4.4M records with at least one subject (as defined above)
*  2.4M records with more than one subject
*  2.0M records with exactly one subject
*  2.6M records with zero subjects

The records with the most subject headings tend to be collections of stuff (theses, photos, etc). Our local standout is the [Dept. of Medicine and Surgery (University of Michigan) theses, 1851-1878](http://mirlyn.lib.umich.edu/Record/004078801) with 208 subject entries. 14 records have at least 30 subject entries. 

### What it means

Gee, lady, I don't know.

One way to look at it: suppose you're considering defining subjects in this way, and making them "hot" in the catalog interface. For our data, 2/3 of records would have either no subjects or a subject that found only the record you're at. So...think again. 

In real life, we index lots of possible subject fields, and we additionally index the $$a as well as the whole string, so ours are a little bit more useful. A little.
