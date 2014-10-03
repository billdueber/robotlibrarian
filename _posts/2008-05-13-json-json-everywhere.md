---
title: "JSON, JSON everywhere"
date: 2008-05-13
tags: "api, datastorage, json"
layout: post
slug: json-json-everywhere

---

Via Ajaxian, just saw an announcement for [Persevere](http://sitepen.com/labs/persevere.php), a network-centric,
JSON-based generic storage engine. It features:

 * A REST-based interface over regular old HTTP
 * JSON as the native data going in and out, including circular references and such
 * Search interface based around [JSONPath](http://goessner.net/articles/JsonPath/)
 * RPC interface based on [JSON-RPC](http://json-rpc.org/)
 * Seemingly buzzword compliant across the board

I've been thinking about these sorts of servers a lot lately ([couchdb](http://incubator.apache.org/couchdb/) and [strokedb](http://strokedb.com/) are two others) in the context of the "not-the-catalog" data we track here at the library.

For some stuff, clearly we need the power and speed of a real database. That power and speed isn't free, though -- you have to set up the tables, map relationships, build an interface on top of it, etc. While it's not rocket science by any stretch of the imagination, it's a lot of screwing around and involves a few levels of security and has a friendly red sign on the door that reads "Programmers only, please."

For other data, though, a structured or semi-structured data store based on a plain text format like JSON would be great.
Since everything is a URL, we can handle security at the HTTP-auth/authz level. Library hours, lists of databases we subscribe to, staff directory data -- these are data that could, if we wanted, be moved into a generic store like this.

The exciting stuff comes when you stop thinking about traditional database applications and think more in terms of having a data storage endpoint that pretty much anyone with a modicum of knowledge and authorization could throw stuff into. Want to build your own tagging system? Your own "My Shelf?" How about a comment form that straddles the edge between "email me the results" and "ask someone to hook me up to a database"? Or a javascript library that automatically takes survey submissions and sticks them into a system like this?

This is the flip-side of my [last post](http://robotlibrarian.billdueber.com/psst-were-not-printing-cards-anymore/). We're not talking about hard-core, multiply-linked, core-business metadata. For that, we need ridiculously smart people figuring out how to best leverage the, say, 8 million MARC records we've got lying around. But for other stuff...this seems really, really cool.
