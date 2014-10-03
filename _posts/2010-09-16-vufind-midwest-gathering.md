---
title: "VuFind Midwest gathering"
date: 2010-09-16
layout: post

---

A couple weeks ago, representatives from UMich (that'd be me), Purdue, Notre Dame, UChicago, and our hosts at Western Michigan got together in lovely Kalamazoo to talk about our VuFind implementations.

Eric Lease Morgan [already wrote up his notes](http://www.catholicresearch.net/blog/2010/09/vufind-midwest-users-group-meeting/) about the meeting, and I encourage you to go there for more info, but I'll add my two cents here.

So, in light of that meeting, here's what I'm thinking about VuFind of late:

* None of us are running VuFuind 1.0 as released with full catalog data. Eric has a [special purpose portal](http://www.catholicresearch.net/) running the current code over an aggregated special collection and hasn't done much to the underlying PHP. The rest of us were running heavily modified versions of RC1. An issue we had in common was that the changes from RC1 to RC2 to 1.0 release were so significant, including some complete architectual change (some based on the stuff I've done with [mirlyn](http://mirlyn.lib.umich.edu/)) that the effort required to get up with 1.0 would be no less significant than the effort to switch wholesale to something else (e.g., [Blacklight](http://blacklightopac.org/)).

* A point that I made that was echoed by others is that we need to remember that these new discovery systems are all just thin wrappers over [Solr](http://lucene.apache.org/solr/). They basically have two jobs: to get a query and format it in a way that Solr can handle, and then to take the Solr results and display them. There's some sugar on top of that (exporting, tagging, etc) but that's really it. The heavy lifting is all done by your indexer ([Solrmarc](http://code.google.com/p/solrmarc/) for most, although watch this space for my announcement of my JRuby-based stuff today) and Solr itself. *It's not a hard problem*, although it is occasionally a messy one.

* VuFind has, in my mind, fundamental architectural issues mostly based on the inability to easily separate local code from core code. A re-architecture to base everything on subclasses of the core code would help, but at some point you start to run up against fundamental limitations of PHP and Smarty to do things cleanly. Without the ability to update core code and know it won't affect your local code, there's no good way to keep on track with the trunk of the code and do upgrades; for the same reason, it's almost impossible to send changes back to trunk.

* Coupled tightly to the architectural issues is the lack of tests. The code is potentially very brittle; there's no good way to know if you're breaking anything until you notice it's broken. It's not at all clear how to write good tests for the code, because there's a lot of inter-dependencies.

* The second big problem is one of community; to wit, there isn't much of one. There are some active players, and there's what seems like a [great conference](http://vufind.org/schedule.php) going on right now, so this may change.  But -- especially because of the technical difficulties in contributing local changes back --VuFind could use a benevolent dictator, someone who has organizing and administrating VuFind *be a part of his/her job*. The last bit is important.


All of these are surmountable issues. The reason they're at the top of my head, of course, is that the Blacklight community has, in many ways, already taken care of most of them.

If I were starting from scratch tomorrow, we'd already decided to do something locally, and I could convince my systems people to run a ruby implementation (I like JRuby myself), I'd go with Blacklight. If we were already looking at something like Summon, I'd take a hard, hard look at the build-vs-buy numbers. Summon and Primo both give you APIs to program an interface against, and boy, it might be worth the effort to do so and leave everything else alone.
