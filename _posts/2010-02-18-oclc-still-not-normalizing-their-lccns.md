---
title: "OCLC still not (NO! They are!) normalizing their LCCNs"
date: 2010-02-18
layout: post
slug: oclc-still-not-normalizing-their-lccns

---

<blockquote>NOTE 2: It turns out that I did find a minor bug in the system, but that in general LCCN normalization is working correctly. I just happened to hit a weirdness with a bad LCCN and a little bug in the parser on their end. Which is getting fixed. So...good news all around, and huge kudos to
Xiaoming Liu for his quick response!
</blockquote>

<blockquote>**NOTE** It strikes me that I haven't seen a case where bad data results from sending a valid LCCN. The only verified problem is one of false negatives. Send a valid lccn, you'll get back either good data or nothing (and the "nothing" might be in error). So, still a big problem, but not as THESKYISFALLING as I imply below.</blockquote>

***

A [long time ago](http://bibwild.wordpress.com/2009/03/11/normalize-your-lccns/), Jonathan Rochkind noted that the OCLC doesn't correctly [normalize their LCCNs](http://www.loc.gov/marc/lccn-namespace.html).

Well, it's not fixed.

I could really, *really* use the xlccn service right about now -- a great web service they provide that, much like xisbn and xissn and the other xXXXX (heh!) services, purports to allow you to put in an lccn and get data back on the item you're interested in.

Except they "normalize" their LCCNs in a way that is not only incorrect, but causes namespace collisions. As near as I can tell, they throw out any leading non-digits and only keep up to the next non-digit.

_The xLCCN service will silently provide no data or **incorrect data** for many LCCN requests!_

An example:

  * (F) Full LCCN is "sn 83011407"
  * (D) First set of digits is "83011407". This is what I think the OCLC is indexing.
  * (N) Correct normalization is "sn83011407"

The problem, of course, is that (D) "83011407" _is itself a valid LCCN_.

  * (F) is associated with OCLC# 47212967
  * (D) is associated with OCLC# 12505148. That's _not the same record_.

So, how do the OCLC services respond?

  * (F) Worldcat search finds correct (probably just doing a string match); xid finds nothing
  * (D) Worldcat finds both correct and incorrect records. The xLCCN service finds *only* the incorrect record, OCLC# 12505148.
  * (N) Neither worldcat nor xid finds anything for the correctly normalized version.

So, what am I supposed to do? Only use the service on LCCNs where the original
and normalized versions are the same and include only digits? Frustrating.
