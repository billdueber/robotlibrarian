---
title: "Does anyone use those prev/next/back-to-search links?"
date: 2010-11-03
---

There's a common problem among developers of websites that paginate, including OPACs: how do you provide a single item view that can have links that go back to the search (or to the prev/next item) without making your URLs look ugly?

The fundamental problem is that as soon as your user opens up a couple searches in separate tabs, your session data can't keep track of *which* search she wants to "go back to" unless you put some random crap in the URL, which none of us want to do.

But let's take three giant steps backwards before we throw a ton of resources at this problem, and ask, "Does anyone use those links"?

### Data from Mirlyn, the University of Michigan OPAC

Here's the data since February of 2010 for [Mirlyn](http://mirlyn.lib.umich.edu/), our library OPAC.

<table style="border: 1pt solid;">
<thead>
<tr>
<th>Action</th>
<th>Count</th>
<th>Pct. of Basic Search count</th>
</tr>
</thead>
<tbody>
<tr>
<td>Basic search (baseline)</td>
<td>1,446,881</td>
<td>100%</td>
</tr>
<tr>
<td>Previous record</td>
<td>1,347</td>
<td>0.09%</td>
</tr>
<tr>
<td>Next record</td>
<td>8,394</td>
<td>0.58%</td>
</tr>
<tr>
<td>Back to search</td>
<td>9,568</td>
<td>0.66%</td>
</tr>
</tbody>
</table>

For what it's worth, I looked at these number by percentage of *sessions* as well, and the numbers come up a little higher -- about 0.8% of all sessions included at least one click of the "Back to Search" button.

Given these numbers, I'm pretty sure I wouldn't put a whole lot of effort into it. In general, next/prev record navigation only makes sense when you have a really, really small number of hits, anyway.

So...why not just disappear the links? I know people will complain, but hopefully our days of doing an enormous amount of work for ...well, some tiny but vocal minority...are past.
