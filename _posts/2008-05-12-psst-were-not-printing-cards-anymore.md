---
title: "Psst. We're not printing cards anymore"
date: 2008-05-12
tags: "code4lib, TALITAS"
layout: post

---

\[From a series I'm calling, "Things About The Library I Think Are Stoooopid", part one of about a zillion.\]

I'm going to wallow in a little bit of hyperbole here, but only a little.

### The problem

Suppose, just for a moment, that you're a computer programmer working anytime in the last twenty years, and someone wants you to set up a data structure to deal with a timeless issue -- how to keep track of who's on which committees in a library.

### If you're a computer person

Easy enough. First off, what's a committee?

**Committee**

 * Committee name (string)
 * Committee inception date (date)
 * Chair (person)
 * Members (set of people)

How about a person?

**Person**

 * Last name (string)
 * First name (string)
 * Email address (email)

Okeedokee. That looks ok so far, but we've got problems.

First off, everyone knows that committee names change. And, everyone also knows that last names can change, preferred first names can change. email addresses change, etc. We need some sort of unique identifier to represent the *abstract ideal* of a particular committee or a specific individual. Let's be lazy and just throw in an integer ID that we'll be careful not to reuse, ever, for any reason.

So, we'll throw that in, and make sure our references are to these unique IDs, not names or whatnot.

That gives us this.

**Committee**

 *  cID (unique integer)
 *  Committee name (string)
 *  Committee inception date (date)
 *  Chair (pID)
 *  Members (set of pIDs)

How about a person?

**Person**

 *  pID (unique integer)
 *  Last name (string)
 *  First name (string)
 *  Email address (email)

And the mapping, of course.

**Committee-Person Mapping**

 * pID (unique integer pointing into the Person table)
 * cID (unique integer pointing into the Committee table)
 * dateTermStarted (date)
 * dateTermEnds (date)


If this seems simple, well, it is. Like I said, the theory is almost forty years old, and common implementations of databases at least twenty. We have well-defined unique keys, special types for dates and email addresses so we can do some sanity checking and order things and so forth, and a very, very simple mapping of people to committees where we keep track of start and end dates just to be complete.

Most importantly, you know what's not here? There's nothing about how to print it out, or what format I'm going to store it in. Those are afterthoughts. They don't matter. Any well-specified data model can be machine-translated into pretty much anything you need.

### If you're writing a library spec

As near as I can tell, the "library" way to write this would be as follows:

**Committee**

\[Let "hus" stand for "hopefully unique string created by ridiculously complex algorithm"]

 * Committee name (hus)
 * Committee inception (string masquerading as a date in any of several formats)
 * Chair (hus)
 * Members
   * person1 (hus) $$b email address (string) $$c start date (date-like string) $$d end date (date-like string)
   * person2 (hus) $$b email address (string) $$c start date (date-like string) $$d end date (date-like string)

Ummmmm...strings. Nothing but strings. Short strings, long strings, fat strings, tall strings. Strings with dollar signs. Strings that look like dates. Strings that contain other strings. And, just for luck, a little bit of hierarchy, where "hierarchy" means "two levels."

If someone's name changes, well, good luck trying to find all the occurrences and fixing them all (and making sure you don't get the wrong John Smith). Good luck parsing out all the dates, which rely not on machine syntax checking but on a whole set of data-enterers trying to follow some sort of rule without making any mistakes. And good, *good* luck getting a list of which committees a specific person belongs to.

### Why I bring it up

One of the most eye-opening talks I heard at [code4lib 2008](http://www.code4lib.org/conference/2008/) was a keynote by [Karen Coyle](http://www.kcoyle.net/) on RDA and its ongoing specification. You can [view the slides or watch the presentation](http://www.code4lib.org/conference/2008/kcoyle) if you'd like.

In it, she makes the point that, when push comes to shove, AACR2 and RDA both ended up being tremendously focused on producing *text strings*.

Whaaaaa??

Was there no one on the RDA committee that had experience with anything even approaching modern data theory?

Of course there was. But the giant weight of history is crushing library data modeling like a skinless grape under a dump truck.

Look, I understand that this is not a simple data modeling problem. I understand that there's a whole set of issues, including a (what I think to be a specious) demand that the cataloged data accurately reflect the actual text in a real, physical object that's sitting in front of you. I'm not so naive as to think this is an easy task.

But anyone who, in the 21st century, approaches the large-scale creation of data without **first and foremost** worrying about machine-parsability, consistent data types with machine-checkable syntax (and even some semantics) and one-to-one mappings between unique objects (an author, an editor, a publishing house, a work) and something that uniquely identifies that object in any reification is....well, I don't know what they're smoking.

We're not printing cards anymore, people.

 * If something is only understandable if a human is reading it, it's not understandable by any modern definition.
 * Punctuation doesn't belong in the description of an object. Ever. Punctuation is a rendering issue. If you're using punctuation, or well-formed strings, instead of descriptive attributes, *you're doing it wrong*.
 * Just because you know your data doesn't mean you know how to model it. Get outside help from the smartest people you can find.

Whew! That felt good!

OK. Rant off.
