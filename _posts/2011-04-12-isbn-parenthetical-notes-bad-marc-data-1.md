---
title: "ISBN parenthetical notes: Bad MARC data #1"
date: 2011-04-12
tags: "bad data"
---

Yesterday, I gave a brief overview of [why free text is hard to deal with](http://robotlibrarian.billdueber.com/why-programmers-hate-free-text-in-marc-records/).

Today, I'm turning my attention to a concrete example that drives me absolutely batshit crazy: taking a perfectly good unique-id field (in this case, the ISBN in the 020) and appending stuff onto the end of it.

The point is not to mock anything. Mocking will, however, be included for free.

### What's supposed to be in the 020?

Well, for starters, an ISBN (10 or 13 digit, we're not picky).

Let's not worry, for the moment, about the actual ISBN and whether it's valid or not.

Wait, no, let's go ahead and worry about it. It's an easy enough script to write, although it takes a while to run.

    8,630,794  Total records
    3,220,666  Total 020a's
        6,498  020a's that don't obviously contain an ISBN
        8,407  that look like an ISBN but fail checksum test:
    ... so 0.26% of the ISBNs have invalid checksums

So, not bad at all, especially considering some of those are known to be bad, but are transcribed dutifully from the actual (mis-)printed book.

A lot of the malformed data (anything from which I can't seem to extract something that looks like an ISBN) is pricing data, and most of it appears in system numbers that are close enough to each other that I presume it was just a bad batch.

### What's goes after the ISBN in the 020?

I'm no cataloger, of course, but it looks to me like the answer is "Something about how the book is bound together, or the publisher, unless you want to put something else there, and then, really, go ahead, because it's not like anyone is ever going to want to parse this out, all we need to do is print cards with it for god's sake."

No, I kid, I kid! The actual rules are in [Library of Congress Rule Interpretation 1.8](http://sites.google.com/site/opencatalogingrules/aacr2-chapter-1/1-8-standard-number-and-terms-of-availability-area), which reads, in part:

> For a hardbound resource, there is no attempt to use a consistent term other than to use one that conveys the condition intelligibly.

I think it's important to read that a second time, because it succinctly conveys the culture in which these rules were devised.

* Don't worry about consistency, because your only reader is human.
* Defer to the cataloger.
* Being complete is more important than being consistent.
* Base your notes on your subjective view of the actual, physical item you're presumed to be holding in your hands.

Interestingly (to me, anyway), it looks like the [OCLC once had a (now deprecated) `$$b` subfield for binding information](http://www.oclc.org/bibformats/en/0xx/020.shtm). Apparently it didn't catch on.

### What did I find?

So, let's pretend I'd like to be able to differentiate between paperback and hardbound books. Probably useful, yes?

I went ahead and took all parenthetical notes from any field in the 020, split them on colon ('cause that seems to be the way they roll) and did some basic normalization:

* Eliminate numbers (so 'vol. 1' and 'vol. 2' count as only one pattern)
* Lowercase everything
* Turn runs of spaces into a single space
* Trim leading/trailing spaces
* Remove any trailing punctuation

I found 1,506,729 parenthetical remarks in the 020 subfields of our catalog.

The top twenty most common entries using those normalizations are:

1. 402537 pbk
2. 387406 alk. paper
3.  99260 v  # _(e.g., "v. 1", "v. 22", etc.)_
4.  82918 cloth
5.  51125 hbk
6.  42036 electronic bk
7.  41360 acid-free paper
8.  38792 hardcover
9.  28913 set
10.  20358 hardback
11.  19160 ebook
12.  16264 paper
13.  15269 u.s
14.  12770 hd.bd
15.  11793 print
16.  10625 lib. bdg
17.  10520 hc
18.   8772 est
19.   7767 pb
20.   7639 hard

The kicker? These are the top twenty of _13,374_ unique parenthetical strings found in the 020 field. Many of them are publishers, or cities, or whatnot, but an awful lot of them are variations on "hardcover" and "paperback."

For example, a quick search for anything that might be "hard" (regexp: /h[ar]{0,2}d/) got me started on a list. Here's just the 90 examples from that list that start with 'h':

> hard \| hard adhesive \| hard back \| hard bd \| hard book \| hard bound \| hard bound book \| hard boundhard case \| hard casehard copy \| hard copy \| hard copy set \| hard cov \| hard cover \| hard covers \| hard sewn \| hard signed \| hard-backhard-backcased \| hard-bound \| hard-cover \| hard-cover acid-free \| hardb \| hard\cover \| hardbach \| hardback \| hardback book \| hardback cover \|  hardbackcased \| hardbd \| hardbk \| hardbond \| hardbook \| hardboubd \| hardbound \| hardboundhardboundtion \| hardc \| hardcase \| hardcopy \| hardcopy publication \| hardcov \| hardcov er \| hardcovcer \| hardcove \| hardcover \| hardcover-alk. paper \| hardcovercloth \| hardcoverflexibound \| hardcoverhardcoverwith cd \| hardcoverr \| hardcovers \| hardcoversame \| hardcoversame as above \| hardcoverset \| hardcovertion \| hardcver \| hardcvoer \| hardcvr \| harddback \| harde \| hardocover \| hardover \| hardpack \| hardpaper \| hardvocer \| hardware \| hd \| hd bd \| hd. bd \| hd. bd. in slip case \| hd. bd.in sl.cs \| hd. bk \| hd. cover \| hd.bd \| hd.bd. in box \| hdb \| hdbd \| hdbk \| hdbkb \| hdbkhdbk \| hdbnd \| hdc \| hdcvr \| hdk \| hdp \| hdpk \| hradback \| hradcover \| hrd \| hrdbk \| hrdcver \| hrdcvr

And that's after eliminating things like places of publication, strings like  "with...", "plus...", "alk. paper", etc.

### "Yeah, but you have to understand that historically..."

Stop hiding behind that.

I understand that at one point in time it probably made sense (to someone at least) to do it this way. I can deal with that.

What I can't accept is that _as I type this_ there's a cataloger doing this in this way. Today. April 2011. Some, what? maybe _thirty years_ since computer-based OPACs became prevalent?

These sorts of problems were recognized _ages_ ago and should have been dealt with. Add a subfield. Invent a controlled vocabulary. Don't worry about the legacy data; it's always going to suck.

But _why are we still producing sucky data???_

### To sum up

The point is that there's a better way to do this stuff. Lots and lots of better ways, in fact. Time I spend dealing with crappy data is time I _don't_ spend making relevancy raking better, or building a better command language search option for my librarians, or working on ways to get a decent "more like this".

The need is both dire and urgent; the latter because sooner or later we're going to have to go to a "two state solution" with traditional MARC21 for many of our records and whatever comes next (RDA?) for the newer stuff. And every day we wait, that first category grows, and the growth rate keeps increasing.

And then there's serials. Don't talk to me about serials.
