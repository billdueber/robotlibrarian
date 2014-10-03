---
title: "Why RDA is doomed to failure"
date: 2010-04-23
layout: post
slug: why-rda-is-doomed-to-failure

---

[Note: edited for clarity thanks to rsinger's comment, below]

Doomed, I say! DOOOOOOOOOOMMMMMMMED!

My reasoning is simple: RDA will fail because it's not "better enough."

Now, those of you who know me might be saying to yourselves, "Waitjustaminute. Bill doesn't know anything at all about cataloging, or semantic representations, or the relative merits of various encapsulations of bibliographic metadata. I mean, sure, he knows a lot about...err....hmmm...well, in any case, he's _definitely_ talking out of his ass on this one."

First off, thanks for having such a long-winded internal monologue about me; it's good to be thought of.

And, of course, you're right on all counts. I don't know what I'm talking about in any of those realms.

And yet I'm still willing to make a strong statement?

Yes. I am. Here's why.

[Oh, and if you're convinced I'm wrong -- please say so. I'd *love* to be wrong about this.]

### First, an assertion

The purpose of any bibliographic metadata is to facilitate three things:

*  **Description/Identification**. If you know what you want, does the metadata give you enough information to determine if the described item is what you want? Alternately, if you're holding an item (or an alternate metadata representation of it), can you find the record that describes it?
*  **Machine finding**. Can a machine, given a good-enough query, find a work via a search of the metadata?
*  **Machine grouping**. Given the metadata, can a machine help a person find items "like this one"?

Take issue with one or more of those statements. I don't care. The point I'm really trying to make is that any standard that doesn't put *unmediated machine reasoning* at the forefront of what the metadata needs to support is living in a deep, deep hole.

Computer cycles are pretty cheap, and programmers are pretty smart. We can figure out how to do useful things with virtually any data, but only if we can reliably get at those data.

### Getting 75% of the way there

Three-fourths of the problem can be addressed with one simple concept.

**A solid equality relationship**.

By this I mean that "=" had better damn well mean "equal," as opposed to "probably the same,  but there might be other representations, too." If I want to say "A = B" (where A and B are authors, or works, or subjects, or anything that can be nailed down) there's better be no false positives and no false negatives. Ever. MARC's use of "hopefully-unique strings" is ridiculously insufficient in the modern era.

RDA does pretty well with this, with URIs for appropriate concepts, so that's good.

### What's wrong with it?

Well, it's gonna cost money to access the spec, for starters. That's just dumb.

But it's also not flexible/extensible enough. It's true that I'm not a
cataloger. I do have an MS in computer science, though, and there is stuff in
the various versions of the RDA spec which lead me to believe that the
committee desperately, **desperately** needed some hardcore geeks on it.
Computer science has basically done nothing but develop methods for
abstraction and composition for *decades*, and that isn't reflected enough
here.

Language such as, "If it is determined that a mechanism for providing a direct
link between a note and the instance of the element to which it relates is
required,..." worries me. *if*? **IF**????? That's not a spec. That's a guideline. Nail it down, for god's sake. When is it appropriate or inappropriate? How do you add links to multiple (but not all) instances of the element?

The spec also seems to describe at least half a dozen kinds of titles. One of these is "Abbreviated title." Do we really want an abbreviated title? No. We want a title with an "abbreviated" modifier, so we can use that same modifier for, say, a corporate name or publisher or anything else. [Note: see rsinger's comment below, indicating this was a piss-poor example on my part.]


### Well, sure, but it's still better than the AACR2!

[This section updated to disabiguate my use of 'MARC' when I really meant 'AACR2 as commonly talked about in term of MARC tags']

Of _course_ it is. It's just not *better enough*!

We're not just talking about writing a spec. We're talking about replacing *every single tool in the library toolchain*, from the ILS to editing software to OPACs to scripts that keep it all put together. We'll be asking programmers to learn new skills and new ways of thinking, vendors to produce functional software for untested data formats, and catalogers to essentially take their whole brain out of their heads and get a new one.

But that, frankly, is the *easy* part. The entire culture of the library is built around AACR2 concepts and MARC data structures. The thought processes, nomenclature -- *everything* sometimes feels as if it's built around three-digit tags. The majority of the (crucial!) specialized vocabulary librarians, and experts and specialists, use to communicate with each other is directly or indirectly tied to MARC

So, yeah, RDA is a hellofa lot better than AACR2/MARC. But in my view, it's not better *enough* to justify all the pain. Switching is incredibly, astoundingly expensive both in terms of cost and in terms of the devaluation of institutional knowledge. We can't do it every few years. We need to be damn sure we're getting it right.
