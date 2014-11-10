---
title: Help me test yet another LC Callnumber parser
tags:
date: 2014-01-30
layout: post
slug: yet-another-lc-callnumber-parser.html
---

Those who have followed this blog and my code for a while know that I have a [long](http://robotlibrarian.billdueber.com/normalizing-loc-call-numbers-for-sorting/), slightly [sad](http://robotlibrarian.billdueber.com/enough-with-the-freakin-lc-call-number-normalization/), and borderline [abusive](https://github.com/billdueber/lib.umich.edu-solr-stuff) relationship with Library of Congress call numbers.

They're a freakin' nightmare. They just are.

But, based on the premise that Sisyphus was a quitter, I took another stab at it, this time writing a real (PEG-) parser instead of trying to futz with extended regular expressions.

The results, so far, aren't too bad.

The gem is called [lc_callnumber](https://github.com/billdueber/lc_callnumber), but more importantly, I've put together a little heroku app to let you play with it, and then correct any incorrect parses (or tell me that it worked correctly) to build up a test suite.

So...[Please try to break my LC Callnumber parser](https://lccparser.herokuapp.com/)!

[Code for the app itself is [on github](https://github.com/billdueber/lccparser); pull requests for both the app and the gem joyously received]
