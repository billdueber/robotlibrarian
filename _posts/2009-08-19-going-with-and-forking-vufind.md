---
title: "Going with and \"forking\" VUFind"
date: 2009-08-19
layout: post
slug: going-with-and-forking-vufind

---

*Note: This is the second in a series I'm doing about our VUFind installation, [Mirlyn](http://mirlyn.lib.umich.edu/). Here I talk about how we got to where we are. Next I'll start looking at specific technologies, how we solved various problems, and generally more nerd-centered stuff.*

When the [University Library](http://www.lib.umich.edu/) decided to go down the path of an open-source, solr-based OPAC, there were (and are, I guess) two big players: [VUFind](http://vufind.org/) and [Blacklight](http://projectblacklight.org/).

I wasn't involved in the decision, but it must have seemed like a no-brainer. VUFind was in production (at [Villanova](http://library.villanova.edu/Find)), seemed to be building a community of similar institutions around it (e.g., Stanford), and was based on a technology stack we had some experience with (PHP). Blacklight seemed to be just getting off to a fitfull start, and its Ruby stack was at that time an iffy proposition (this was before any sort of major adoption of [Passenger](http://www.modrails.com/) or [JRuby](http://jruby.org/)).

As I write this, things have flipped around a little. Andrew Nagy, the principle architect of VUFind, left Villanova for Serial Solutions and VUFind stopped being his primary focus. The Blacklight community decided to go with a major reorganization of the code to make it easier to deploy, which resulted in a flurry of refactoring and improvements and folks generally thinking things through really well. Stanford just flipped the switch from their VUFind to a Blacklight installation, and as I pointed out, the Ruby deployment options are more stable and less resource-hungry than they were back then. If the decision were being made today, it would be a much more complex analysis.

But anyway, the decision was made, and Tim Prettyman and I were tapped to do most of the hardcore nerd work to make it suitable for our environment.

Right away, I found things that would need some pretty major revision. The user model was based on a local database of logins (we use [cosign](http://www.umich.edu/~umweb/software/cosign/)), even moderately-long search strings would crash the thing, cookies were being used instead of sessions and hitting the 4K limit, search specification were hardcoded in the PHP, and lots of the UI elements didn't actually have working code behind them (RSS feeds, endnote export, spellcheck, etc).

So, I dug in and started learning PHP and Smarty and refactoring/rewriting/rearchitecting the crap out of it. One of the first things I did was to extract the search specification -- the mapping of, say, a 'title' search to a weighted search of six or seven actual Solr fields -- into a yaml file so we could mess around with it more easily than modifying the giant case-statement in the PHP code. I built a patch against the then-current revision, filed it as a bug, and sent email to the list.

And nothing happened. That patch is still sitting there, in fact. Maybe I'm the only one that thinks it's useful. But in any case, there was no discussion of it, no one *rejected* it. It just sat. Sits. Whatever.

I could have asked for write access to the repository, but I didn't. I saw a few other patches get submitted and met with yawns all around, and started looking more closely at the list and saw pretty much no one doing anything with the then-current code base, and frankly kind of gave up. The folks that I knew were working actively on implementing VUFind -- us, Australia and Alan Rykhus at  [MNPals](http://www.mnpals.net) -- were all working from very different code bases, which made our ability to share code very limited. Any sort of official work on VUFind seemed to have slowed to a near standstill (based on svn checkins), and almost no one else seemed interested in submitting patches. After a while, we stopped, too.  

So, we didn't really *fork* VUFind. We just rewrote much of it and stopped trying to generate interest in our changes. The *right* thing to do would have been to either grab the bull by the horns, or do an actual fork of the project. But we didn't feel as if we had time to shepherd a project of this size, and after many, many (*many*) discussions, decided to just do our thing. I assume that's what everyone else has done, too, since I see plenty of differences in how things work at the different sites.

As it stands, the wiki shows a [good handful](http://vufind.org/about.php) of libraries live with VUFind, and a bunch more marked as being in "beta." I don't know if what we're running [Mirlyn](http://mirlyn.lib.umich.edu/) on is still enough VUFind to be called VUFind. Probably. The basic structure is the same, the search syntax as exposed in the URL is the same. The plumbing underneath is changed in a lot of ways, and I like to think the flow of control makes a little more sense now.

In real life, of course, it doesn't matter where you draw the line. Our code is far enough removed from the svn repository now that we're essentially going it alone.

That doesn't bother me.

The reality is that we've taken control of the UI and learned what we need to know about using Solr with our data. If I need to change the backend -- to Blacklight, to a newer VUFind, to anything -- my users need not ever know, other than to notice that things are a little bit better. If we end up moving to a release-quality version of VUFind, there's almost nothing I can't reuse if it makes sense.

We've also learned a lot. Solr, obviously, and how to write text filters for it and push it around just a little bit.  Solrmarc, too. But we've also taken a hard look at data normalization in ways we haven't before, and decided how we're going to output to Refworks, and to email, what kinds of searches we want to offer, where we have collisions in ID namespaces (OCLC & ISSN, I'm looking at you).

We've discovered issues and problems with our data we'd have never seen otherwise, and started up whole sets of conversations about OPAC issues that used to languish for lack of a reification for reference.  The ability to actually (try to) implement the collective intelligence of the library and embody it in a public-facing system is a rush compared to fighting with the ILS.

The system has tons of problems still, starting with underlying templates that will make you a little sick if you do a "view source" and going right through my call number search not working for some edge cases. But that stuff will get cleaned up as we get a little downtime from adding new features, and there are elements of the new backend code that could be useful to others once I clean them up and remove local dependencies.

I'm not sure when, if ever, we'll start thinking of ourselves as part of the "VUFind community" again. The heavy intellectual lifting about how to organize what is essentially a front-end for Solr doesn't seem to be happening on the VUFind list. And to be honest, I'm not sure it should be. Solr is the real engine. Solrmarc is, for us right now, an important piece. Data normalization, translation, workaround for crappy data, and the basic information theory of a faceted search system are all independent of the particular middleware you're using to grab Solr results and throw them up on the screen.

So, what we have is good for us, for now, and we're continuing to learn how to move forward. And I've been able to get bug reports and say, "Thanks, Fixed" fifteen minutes later and get warm fuzzy feelings that don't usually accompany, "Thanks. I'll put a request in at Ex Libris' online ticket system".

Next time: using and abusing Solr for data normalization.
