---
title: "Corrected Code4Lib slides are up"
date: 2011-02-15
layout: post

---

...at <a href="http://robotlibrarian.billdueber.com/wp-content/uploads/2011/02/dueber_lightning_c4l11.ppt">the same URL</a>.

I was, to put it mildly, incredibly excited about <a href="http://code4lib.org/">code4lib</a> this year because, for once, I thought I had something to say. And I did have something to say. And I said it. But it was wrong.

I presented a bunch of statistics drawn from nearly a year of <a href="http://mirlyn.lib.umich.edu/">Mirlyn</a> logs. The most outlandish of my assertions, and the one that eventually turned out to be the most incorrect, was that some 45% of all our user sessions consist of only one action: a search.

Unfortunately, I'd missed a whole swath of things I should have excluded. I'd remembered robots and stuff coming in from our link resolver and so on. I hadn't counted on having to fight my own stupidity.

In short: <a href="http://catalog.hathitrust.org/">catalog.hathitrust.org</a> and <a href="http://mirlyn.lib.umich.edu/">mirlyn.lib.umich.edu</a> share a common code base, as well as a Solr backend. I was correctly excluding all the HathiTrust stuff from my stats <em>except for simple searches</em>. What I ended up with was a whole lotta sessions with nothing in them but that search. Luckily, I noticed waaaay too many people coming in via the HathiTrust site (which I <em>know</em> doesn't have a link to Mirlyn) and did more digging.

The slides have been updated with correct numbers. Luckily, even though the adjustment was pretty extreme, I don't think many of my conclusions are invalidated, especially given <a href="http://www.lib.umich.edu/files/services/usability/MirlynSearchSurvey_Feb2011.pdf">corroborating evidence from an extensive survey conducted by our usability team</a> (PDF). They conclude, among other things, that known-item searching is prevalent and relevancy raking is important across task boundaries.

The basic stats from the powerpoint, for those who don't want to read all my notes:
<ul>
	<li>17% of all sessions have one action: a search</li>
	<li>In only 28% of all sessions does the user see the Record View</li>
	<li>75% of all logged actions that target an individual record (see the full record view, look at extended holdings, etc.) happen with a record in the top 6 search results</li>
	<li>7% of sessions involve a user adding a facet</li>
	<li>2% of sessions involve a user exporting records</li>
</ul>
