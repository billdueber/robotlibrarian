---
title: How good/bad is MARC data? The case of place-of-publication
layout: post
date: 2014-11-10
---

I complain a lot about [the MARC format](/2010/04/data-structures-and-serializations/),
[the way people put data in MARC records](/2011/04/why-programmers-hate-free-text-in-marc-records/),
the [actual data themselves I find in MARC records](/2011/04/isbn-parenthetical-notes-bad-marc-data-1/),
the [inexplicably complex syntax for identifiers](/yet-another-lc-callnumber-parser/) and, ironically, 
[attempts to replace MARC with something else](/2010/04/why-rda-is-doomed-to-failure/).

One nice little beacon of hope was when I found that only roughly 0.26% of the ISBNs in the UMich catalog 
have invalid checksums. That's not bad at all, and it's worth digging into other things about which
I might be likely to complain before I make a fool of myself.

[Note: there *will* be some complaining at the end. I promise.]

One of my recent charges was to try to put in place a better place-of-publication filter in the catalog.
*Place of Publication* is most formally dictated by the (poorly-named, since it includes states/provinces) [Country of Publication](http://www.oclc.org/bibformats/en/fixedfield/ctry.html)
code in the 008 fixed field. This one-, two- or three-letter code that is then translated into a place name 
via a [mapping provided by the LoC](http://www.loc.gov/standards/codelists/countries.xml). Like most important pieces of data,
the place of publication can appear in a few different places in a valid MARC record -- because the searching is half the fun! --
but we decided to just stick with the
008 for the catalog search.

Of course, the name of a place of publication may have changed since the actual publication. 
Historically speaking, borders have been remakably consistent over the last half of a century
or so, but there are still changes (fall of the Soviet Union), splits (the former Czechoslovakia)
and merges (Germany). 

## Focus on validity

So, there are roughly a bazillion ways one could try to slice and dice the data to figure out what the most
accurate textual representation of a place name should be for a given record. More cut-and-dry is a simple question: 
how many of the 008s have a valid (current or obsolete) place-of-publication code in them?

I ran an analysis of all the 008s in all the records in the [University of Michigan catalog](http://mirlyn.lib.umich.edu/),
which helpfully includes all the HathiTrust holdings as well, so we're getting a nice cross-section of institutional records.

Here's what I found, in round numbers

|           | Total      | Pct. of Total |
|:----------|-----------:|--------------:|
| All records | 12M | 100% |
| Invalid 008 | 1900 | 0.15% |
| Valid code | 11.6M | 96.6% |
| Unknown place-of-pub | 381k | 3.1% |
| Invalid code | 27k | 0.2% |

["No place-of-pub" includes both records with no data in the 008 and those with the code '  x' which explicitly indicates "Unknown"]


## Results: pretty good!

Given much of the data I've worked with over the years, this strikes me a stunningly good. Of course, in the case of a place as 
big as UMich, that means we've still got about 408k items about which we have no good place-of-publication information, but 
as a percentage, it's small enough that I'm happy to live with it. 

I was, admittedly, a little put out by the fact that we have records in which the 008 fixed field -- which is pretty important, as these things go -- was 
just plain invalid (including 176 just plain *missing*). You'd think that the ILS software would reject things like that, but,
as in almost all cases when you think the ILS should do something smart, you'd be wrong.

## And now, the complaints

Of course, all we know is that the codes are (or were) valid -- not whether or not they're *accurate*. 

There are two obvious problems:

* Some rocket scientists at some point decided that the code 'ai', which had been used to represent Anguilla, should now be
used to represent the Republic of Armenia. As if that weren't enough to make you slam your head into a brick wall,
the change is based on the date of *cataloging*, not the date of publication, so there's no way for me to know which
country is supposed to be indicated. It looks like this was to try to keep the first two letters of codes from the old Soviet Union
the same one it fell apart, but c'*mon*, people! (Note that Anguilla is now 'am', because of the ...ummmm...."m" in it's...er...nevermind.) We don't have
many records with that code, but this is the sort of blatent disregard for simple data integrity that drives me *crazy*. 
* A presumably different set of rocket scientists (once NASA downsized, those folks were *everywhere*) at various points in time and 
at various locations decided that the place of publication on a reproduction (say, a microfilm) should be the place the *reproduction*
was created. So, a microfilm of *The New York Times* that happened to be created in Ann Arbor, MI is coded as 'miu', for Michigan. 

The latter, of course, is designed to serve those people studying where microfilms were created at the expense of people who
want to, you know, find things actually published in a particular location. I'm sure *all three* of the people in the country who want to know the
former are forever grateful. 




