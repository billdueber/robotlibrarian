---
title: "TicTocs: Give us a file! Pretty pretty pretty please!"
date: 2009-02-02
layout: post
slug: tictocs-give-us-a-file-pretty-pretty-pretty-please

---

For those who haven't heard, <a href="http://www.tictocs.ac.uk/">ticTOCs</a> is a service that provides web-based access to a database of Journal RSS/Atom Table of Contents feeds. Awesome.

In their blog at <a href="http://tictocsnews.wordpress.com/">News from TicTocs</a>, a post titled <a title="Permanent Link to I want to be completely honest with you about ticTOCs" rel="bookmark" href="http://tictocsnews.wordpress.com/2009/01/27/i-want-to-be-completely-honest-with-you-about-tictocs/">I want to be completely honest with you about ticTOCs</a> notes that:
<blockquote>As for the API - yes, we’ve been asked this several times, and the answer is that it is currently being written and should be available very soon.</blockquote>
That's great, but writing in a comment on that post (after logging in with a very, very old OpenID -- I used to have a blog named <em>Opachyderm</em>, a name which I thought was insufferably clever), I noted that we don't need an API right away.

What we need is a text file.

Simple. Tab-delimited. TicTocID,Title,URL,issn,eissn. Update it every night.

That's all we need.

We can do the rest. Put it in the OPAC. Stick it on our SFX pages. Not screw around with Javascript/AJAX calls when the data we need are (relatively) static and (absolutely) simple.

Someone needed to put a web interface on those data, and the one provided at ticTocs is really nice. I'm glad it's there.

And I can't tell you how much I applaud the JISC for starting this project and getting vendors on board. That's always the hard part -- participation and standardization. They're doing it, and I couldn't be happier.

But these data are incredibly valuable,  and their value is currently limited because they're boxed up.

Spreading these data far and wide is good for scholarship, and I can't imagine the case that could be made showing it's better for JISC to keep them at a single endpoint.

The knee-jerk reaction is always, I know, to keep things behind a wall, even if it's a short wall. "Things will get out of sync if people have their own copies." Or, "We'll provide whatever access you need, as fast as you need it, honest." Or, "We're going to be providing value-added services on top of the data."

It's all true. Things will get out-of-sync -- but that's going to happen whether you encourage people to not cache results or not. And I don't doubt for a moment that the API provided will be great. And of course you'll be in a position where you can provide value-added services.

But so can the rest of us.

I've run into this myself. I fear...well, let's be honest. I fear providing a service, having the data stripmined, and then having no one appreciate the front-end I put on it. I do this job for the fame, not the fortune. Obviously.

But I'll never provide services as fast as me plus three hundred other geeks, all responding to different situations and servicing different patrons.

So...provide an API. Start simple: a single call named <em>getCurrentTextFile</em>. Or maybe add <em>getCurrentTextFileGzipped</em>. It's only ten-thousand lines of text, probably less than 75k gzipped up. I promise to call it every night about 3am local time so I'm up-to-date.

So....pretty please? With sugar on top? My catalog is waiting. So is my SFX install. And our list of ejournals. And our subject guides. And lots of pages on our website. And our pre-packaged OPML files to offer students and professors. And a thousand yet-to-be-devised services as well.

Pretty pretty pretty please???
