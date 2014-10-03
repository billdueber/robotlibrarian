---
title: "Please: don't return your books"
date: 2013-02-12
---

So, I'm at [code4lib 2013](http://code4lib.org/conference/2013) right now, where side conversations and informal exchanges tend to be the most interesting part.

Last night I had an conversation with the inimitable [Michael B. Klein](https://twitter.com/mbklein), and after complaining about faculty members that keep books out for _decades_ at a time, we ended up asking a simple question:

> How much more shelving would we need if everyone returned their books?

Assuming we could get them all checked in and such, well, where would we put them?

I'm looking at this in the simplest, most conservative way possible:

* Assume they're all paperbacks, so we don't worry about how thick a cover is (cover width = 0)
* Assume items for which we don't have page count information are "average"

### Starting data

What's my current situation at Michigan?

* Total bibs: about 10M (but that includes a bunch of HathiTrust items and other electronic-only items that could never be checked out)
* Total items checked out right now: 162,080


The first problem I run into is that I don't know how many pages are in a given book. Well, in theory I can look in MARC field 300$a, and it will tell me.

### Finding the number of pages in a book

I went through a recent dump of all our records and pulled out page counts from the 300 (those that matched the regular expression $$a\d+\s+[pP]\.).

Problem solved, right? Well, kind of

* 3,085,433 total bibs with page count data (about 30%)
* 40,872 checked out items with page count data (about 25%)

OK, so I don't have data for everything. Plus, some of those are multi-volume works that list the total page count, even though only a single volume may be checked out.

We'll have to drop down into statistics:

* Average number of pages in a checked-out item: 270
* Median number of pages in a checked-out item: 244

The median is lower, so we'll go with that. Being conservative, remember?

### Bringing it all together

Obviously we need to make a lot of assumptions.

* All paperbacks (== no space allowance for covers)
* 244 pages per item (the median of checked out items for which we have data)
* Pages = 244 * 162,080 = 39,547,520 pages

### So...what's the damage?

But how to do the calculation?

It turns out that simply googling [book spine width calculator](https://www.google.com/search?num=30&amp;hl=en&amp;safe=off&amp;tbo=d&amp;noj=1&amp;site=webhp&amp;source=hp&amp;q=book+spine+width+calculator&amp;oq=book+spine+widt) a few come up.

I picked one and input 39,547,520 pages and assumed 50lb paper (the lightest paper in the tool).

**Total width: 77,241.25 inches, or 6437 feet, or 1.22 miles**

### 1.22 miles???

Well, we had a lot of assumptions,but most of them were pretty conservative. And I have no idea if the book spine calculator is at all accurate.

But...it's gonna be a big number no matter what. Add in that many of them are hardcover, and this seems like a pretty good guess at a lower end.

### What is this good for again?

Oh, nothing at all. Just a little fun while I'm at code4lib.

### Next steps

Well, the best next step would be to walk away. This is a huge waste of time.

But...we could look in the 020s for a hint of whether it's hardcover or paperback (which is [really hard](http://robotlibrarian.billdueber.com/isbn-parenthetical-notes-bad-marc-data-1/). And maybe try to figure out if multiple volumes of a multi-volume work are all checked out and take that into account.

But really: this is enough for me. Whether Michael wants to pursue it further on his own, well, that's up to him.
