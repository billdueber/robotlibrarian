---
title: "Rolling out UMich's \"VUFind\": Introduction and New Features"
date: 2009-08-14
---

<p>For the last few months, I've been working on rolling out a ridiculous-modified version of <a href="http://vufind.org/">Vufind</a>, which we just launched as our primary OPAC, <a href="http://mirlyn.lib.umich.edu/">Mirlyn</a>, with a slightly-different version powering <a href="http://catalog.hathitrust.org/">catalog.hathitrust.org</a>, a temporary metadata search on the HathiTrust data until the OCLC takes it over at some undetermined date.</p>

<p>(Yeah, the HathiTrust site is a lot better looking.)</p>
 
<p>[Our Aleph-based catalog lives on at <a href="http://mirlyn-classic.lib.umich.edu">mirlyn-classic</a>) -- I'll be interested to see how the traffic on the two differs as time goes on.]</p>

<p>I'm going to spend a few posts talking about how and why we essentially forked vufind, what sorts of modifications I made, and what technologies I hope to extract from our implementation that may be useful to the wider library community. And, I'm sure, a lot about why I hate Solr, why I love love love Solr, why I hate PHP, and why I love...er...no, I still hate PHP.</p>

<h2>Credit where it's due</h2>

<p>And... a little credit where it's due. I did a lot, but I didn't do it all.
  I probably didn't even do most of it. Half the effort, including all the heavy Aleph lifting -- from getting the MARC out with all the filters and expansions we needed, to pulling holdings in real time, to grabbing a patron's current checked-out items and holds, to fighting the inevitably-scarring battle with ILL -- was done by Tim Prettyman. Suzanne Chapman lent her expertise to make it a lot less ugly and more usable than it once was (you can see her talents more strongly expressed at the <a href="http://catalog.hathitrust.org/">HathiTrust catalog</a>). And a whole horde of librarians were tapped by my boss, Jon Rothman, to try to figure out how to deal with the MARC data and facets and everything else that required a much deeper understand of our data than I possess. </p>


<h2>Non-stock user-facing features</h2>
<p>In the next post, I'll start with a look at how and why we changed the backend and what I'd do differently if I were starting from scratch. But right now, a quick list of the user-facing stuff that you might find interesting.</p>
 
<ul>
  <li>Email and export searches and search results, as opposed to just individual records.</li>
  <li>Working endnote and refworks export.</li>
  <li>Multi-select on the advanced search (e.g., pick two languages to get English OR German).</li>
  <li>Publication date-range searching (with date-added-to-catalog searching coming soon).</li>
  <li>A "sticky" institution selection, so each campus can choose to default to searching just their own stuff. We sniff IPs to set a default, too.</li>
  <li>A "call number starts with" search based on semantics for LC searches (e.g., searching on CA11 won't find CA1105), with call number range searching in testing now.</li>
  <li>Contracted holdings for long lists of serials (see, e.g., <a href="http://mirlyn.lib.umich.edu/Record/000637680">Nature</a>).
    <li>[Coming soon] Selecting records to a temporary set, which can be manipulated <em>en masse</em> (sent to Refworks, etc.). I'll be hooking this up to <a href="http://www.lib.umich.edu/mtagger/">mTagger</a>, our home-grown bookmarking and tagging tool, later on.
</ul>


<p>Of course, I also broke some things. I haven't added back in Search History, but will do so when I've got a couple hours. "Search Within" will make a comeback soon, too, but there are usability issues to contend with. And ...for the love of god, don't do a "View Source." It's the ugliest HTML underpinnings I've been associated with since 1993 or so.</p>

<p>All in all, though, it's not bad work, and I'm glad to be able to offer it to our patrons.</p>
