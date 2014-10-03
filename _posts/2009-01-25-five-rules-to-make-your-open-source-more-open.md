---
title: "Five rules to make your open source more open"
date: 2009-01-25
layout: post
slug: five-rules-to-make-your-open-source-more-open

---

[I've noticed that a sure way to get people to look at stuff (as measured by, say, <a href="http://digg.com/">digg</a>) is to include a number. So I did. Five. ]

Over at <a href="http://bibwild.wordpress.com/">Bibliographic Wilderness</a>, Jonathan Rothkind has a great followup to an ongoing discussion on the Blacklight list called <em><a href="http://bibwild.wordpress.com/2009/01/06/how-to-build-shared-open-source/">How to build shared open source</a></em> in which he tackles some of the differences between open-sourcing your code (a legal and distribution issue) and actually making it so someone else can usefully contribute to your code.

The project I'm spending most of my time on right now, <a href="http://vufind.org/">VUFind</a>, is a great piece of functional code but, in many ways, a nightmare in terms of trying to contribute code and abstract out local functionality. This isn't meant as a slam on the main contributor(s) to VUFind -- Andrew, especially, seems to be an almost frighteningly-productive coder -- but my experiences trying to customize the code to our local situation has given me a lot of time to think about how I <em>wish</em> things had been architected.

So, here I give some general rules and some specifics as to What I Wish I Had To Work With.
<h3>1. Abstraction</h3>
<strong>General rule:</strong> Abstract things out as much as makes sense

<strong>Specific rule:</strong> Abstract <em>the living crap</em> out of your authentication scheme.

Look, pretty much everyone with anything worth protecting already has an auth/authZ infrastructure in place. Sometimes an extensive, perhaps multi-institutional infrastructure. One that isn't going to be bypassed without, say, getting fired.

So if you're going to require people to log in, make sure you make that process as abstract as you possibly can, both in algorithm and in code. Have a singleton class that's easily subclassed to represent your user, and call it exclusively. Make sure that your URIs are easily separated into those that require auth and those that don't, for simple use of mod_rewrite or whatnot to redirect to authentication. Make sure it's easy to hook into (or work around) AJAX links that might require authentication that has expired.

And for the love of god, don't stuff username/password information into a cookie if you're doing web work. Use a session and session key. Any auth scheme that <em>I</em> can spoof is no auth scheme at all, because I'm an idiot and not even trying hard.
<h3>2. Configuration files</h3>
<strong>General rule:</strong> use config files for anything local

<strong>Specific rule:</strong> Use a configuration file format that can represent complex data

That's right, I'm looking at you, <em>.ini</em> and <em>.properties</em> files.

Use something like YAML, or XML, or even straight programming-language code (i.e., a file with a PHP hash or a perl hashref or whatnot) that can actually represent, in a logical way, the complexities of the stuff you need to configure. And then, again, have a singleton class that will read that data and expose it in a useful and safe way.

And include a semantics checker if you can manage to write one.  It'll save everyone a load of trouble.

<em>Huge bonus points</em> if your configuration singleton class can read from multiple files, overriding previous (default) definitions with subsequent (local) ones.
<h3>3. Hide subapplications</h3>
<strong>General rule:</strong> Don't force your user to intimately understand every piece of every library/application you include

<strong>Specific rule:</strong> Generate configuration information for sublibraries/applications

This might be a little specific to the project I'm working on now, which uses <a href="http://lucene.apache.org/solr/">Solr</a> as a backend, but I think it applies more generally.

<em>If</em> you're using a non-brain-dead configuration file format, and <em>if</em> you can assume reasonable defaults, <em>then</em> generate configuration files for your user. A low-level extreme of this is the traditional unix <em>autoconf</em>, which essentially allows you to install software without knowing a damn thing about your own system. Which is useful to those of us that don't.

In VUFind, there are three files -- a .properties file that specifies how to map MARC data into  field names,  Solr's <em>schema.xml</em> that describes the structure and behavior of those same fields, and an XSLT stylesheet that pretties the data as it comes out of Solr to make it easier to work with. As you might expect, the overlap in data is about 80% across the three of them, and it would be a bazillion times easier to have a single file that generated all three.

OK. Maybe not a <em>ba</em>zillion, because if it was that easy, I'd have taken a couple hours to write the code to do it already. Let's say just a <em>zillion</em> times easier.

The caveat to this is that you need to either make sure your config file specification is complete enough to encompass everything all the other files might need to know (bad), or that the other config files can import subsections that override your defaults (good).
<h3>4.Testing</h3>
<strong>General rule:</strong> practice test-first (TDD or BDD) development

<strong>Specific rule:</strong> write your code in such a way that it's testable

Look, we all know we should spend the first three weeks writing eight thousand tests to describe every corner of the code. And we all have bosses that will ask, every morning about 10:30am, "So, what do you have that you can show me?"

Not everyone is going to be able to write tests first. That's not right, it's not smart, but it's the way the world works. But at least <em>put in the hooks</em> so someone else can come along and write tests.

Writing tests is one of the easiest ways that a newbie can come along to a project and instantly contribute in a meaningful way. But if you're constantly calling global variables, depending on live database connections and not providing a way to mock them up, or throwing fatal errors if every subsystem isn't present no matter the context, then it's going to be hard to write tests.  So hard, in fact, that not only will you not do it, but neither will anyone else.
<h3>5. Error handling</h3>
<strong>General rule:</strong> provide a sane, hierarchical set of error classes and hooks to catch them as necessary

<strong>Specific rule:</strong> THROW SOME GODDAMN ERRORS!!!!

Don't be an idiot. Things will fail. In the absense of Design by Contract or somesuch, errors will happen. Throw them. Catch them. But at least throw them, instead of letting your code die six hundred lines later with a "Cannot cast null value to string" when you finally get around to trying to print something out.
