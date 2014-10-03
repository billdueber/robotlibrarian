---
title: "Wanted: a better proxy server"
date: 2008-10-02
tags: "TALITAS"
layout: post

---

We in the library world have a problem. We spend a zillion-with-a-Z dollars subscribing to online databases, purchases which presume our ability to make sure only authorized people can look at them. The alternative is to be in breach of contract law, which I've been assured is something we'd like to avoid.

The problem I see is this: <em>The limitations of our proxy server software restrict how we can write contracts with our vendors</em>.

The standard approach is to define two types of access:
<ol>
	<li>By IP address. The person is sitting in front of the right computer (or has hooked up to the right wireless network) and is assumed to be "OK" based on either the location of the computer (e.g., in the library building) or through the nature of the auth/authZ built into the computer's login procedure. We tell our vendors, "Hey," (all vendor-library conversations start with 'Hey') "here's a list of IP addresses that you should allow and associate with us."</li>
	<li>By authenticating with a central mechanism and then sending everything through a rewriting proxy server, thus allowing us to tell the vendor, "Hey. Anything coming through our proxy server is OK. Honest."</li>
</ol>
The venerable EZProxy (now owned by OCLC) has been the solution of choice for libraries for a long time. It does what it does very well.

But I want more. Much more. More more more.

The current model assumes there's exactly one question: <em>Is this person authorized as a UM-Ann Arbor user?</em>

But that's a pretty crude question. Suppose the Business or Law school wants to buy access to stuff for only their students (news flash: they already do)? Or we want to subscribe to a journal but, because it's so esoteric, restrict access to a couple departments to save money. Or recognize when an Ann Arbor faculty member is sitting at a public computer on a different campus but still allow her to get full rights as an Ann Arbor faculty member instead of appearing to be Joe-Random-Dearborn student, a group which has significantly less access to online journals.

Why can't people with roles on multiple campuses get the best of all worlds, getting the least restrictive access possible to a given title  based on all their student/staff/faculty affiliations?

Why can't we negotiate access to given titles (or even articles???) in lieu of course packets (or online reserves), restricting access to only those enrolled in the class?

Here at UMich, we're just starting to get an Enterprise Directory online where we'll actually be able to ask some of these questions. But until we get a proxy server that's smart enough to do something with all the information, it'll just sit there and taunt me.

This isn't an idle question. We already have databases that the Business School subscribes to alone that can only be accessed when you're physically in the B-School at one of the approved-IP-address computers. That's freakin' ridiculous.

Of course, this all presumes that all-or-nothing contracts aren't the best way to go, but shouldn't we at least have the option?
