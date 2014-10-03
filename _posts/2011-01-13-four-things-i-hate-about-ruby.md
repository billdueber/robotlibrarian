---
title: "Four things I hate about Ruby"
date: 2011-01-13
tags: "ruby"
layout: post

---

Don't get me wrong. I use ruby as my default language when possible. I love JRuby in a way that's illegal in most states.

But there are...*issues*. There are with any language and the associated environment. These are the ones that bug the crap out of me.

* **Ruby is slow.** Let's get this one out of the way right away. Ruby (at least the MRI 1.8.x implementation) is, for many things, *slow*. Sometimes not much slower. Sometimes (e.g., numerics) a hell of a lot slower.

Now, there's nothing necessarily wrong with that. For what I do, MRI Ruby is usually fast enough, and JRuby is pretty much always fast enough. But the community response ("Buy more hardware! Programming time is more expensive than CPUs anyway! These are not the droids you're looking for!"), esp. surrounding Rails, is simply annoying. If The Power That Be want to make a *decision* to not worry about improving the performance of the language, well, that's fine then. But to pretend -- or even *insist* -- that it's not at all in issue, well, that's just disingenuous.

* **Version nonsense**. Yes, yes, I understand the historical process that produced a version 1.9.x that's *not backwards compatible* with 1.8.x. But it's dumb. Gems don't often seem much better (including, hypocritically, my own). Versioning -- meaning assigning version numbers in such a way that the underlying semantics are transparent -- doesn't seem to be something The Ruby World (tm) is all that interested in.

* **No relationship between gem names and the modules they contain.** This drives me freakin' crazy. The Perl community does a great job with this. One module per file, one file per module, filenames follow the module names. I know *exactly* what to put after `use` in Perl. In Ruby, what comes after `require` is anybody's guess.

* **Lack of thread-safety**. Look, I get it that the MRI doesn't have real threads. And so maybe there's not a huge incentive on the part of the core folks to make things thread-safe in general. But at least one language construct -- [autoload](http://jira.codehaus.org/browse/JRUBY-3194) -- is just plain broken under real threads, with seemingly little interest in getting it fixed.
