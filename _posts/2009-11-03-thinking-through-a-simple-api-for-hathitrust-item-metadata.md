---
title: "Thinking through a simple API for HathiTrust item metadata"
date: 2009-11-03
layout: post

---

**EDITS**:

  * Added "recordURL" per Tod's request
  * Made a record's *title* field an array and call it *titles*, to allow for vernacular entries
  * Changed item's *ingest* to *lastUpdate* to accurately note what the actual date reflects. This gets updated every time either the item or the record to which it's attached gets changed.
  * Fixed a couple typos, including one where I substituted an ampersand for a pipe in the multi-get example (thanks again, Tod).
  * Added a better explanation of option #4

### Introduction and History

Ages ago, I wrote a simple(ish) little cgi program to get basic item-level data out of what is now [Mirlyn Classic](http://mirlyn-classic.lib.umich.edu/), our OPAC. Soon enough, I was asked to modify it so people could get [HathiTrust](http://catalog.hathitrust.org/) data from the underlying Aleph system, check viewability of the associated items, etc.

It works...kinda...but reflects what I now look back on as blissful ignorance. It doesn't deal at all with serials, and doesn't deal correctly with duplicated records or cases where multiple records have the same (supposedly-unique) identifiers,

We need something better. And I'm hoping comments on this post will result in something better.

### Scope

> Given standard identifiers for a known item, return basic item-level metadata for volumes deposited in the HathiTrust.

I want to keep this *simple*. There will likely be other APIs for other, more complex (or specialized) tasks, linked data for folks who dig that sort of thing, and so on. The goal here is to make something that's *fast* and can help people inline data about HT into their own OPAC or similar system.

I'd also like to get this thing in place, at least the basics, in the next two weeks. Anything longer is self-indulgent.

### Data returned

At the moment, I'm only planning on offering JSON out, unless someone really, really needs something else. Speak up if you're an edge case.

#### Proposed basic return structure

...complete with JSON-illegal comments embedded


~~~javascript

  {
    "records":
      {
        "003384758": // The HathiTrust record id of a matched record
          {
            "recordURL" : "http://catalog.hathitrust.org/Record/003384758"
            "titles":  ["Full, space-joined 245s"],
            "isbns" : ["123456789X"], // any/all ISBNs on this record
            "issns" : [], // any/all ISSNs on this record
            "oclcs" : [], // any/all OCLC numbers
            "lccns"  : ["68001537"], // any/all LCCNs
          },
          ... // any more records that were matched
      },
    'items' :
      [
        {
          "fromRecord" : "003384758",
          "htid": "mdp.39015054407062",
          "itemURL": "http://hdl.handle.net/2027/mdp.39015054407062",
          "rights": "ic",
          "orig": "University of Michigan", // supplying institution
          "lastUpdate" : "20090807" // date of ingest into HathiTrust or last change
          "enumcron" : "An enumeration/chronology, if available" // OPTIONAL
        }
      ]
  }

~~~

#### A quick walk through the proposed return structure

Obviously, there are two sets of items: a list of records that matched the query, and a list representing the union of all items on those matched records.

##### records

For most purposes, people won't care so much about the record-level data unless you're trying to do your own error-checking (possible) or want to link to the catalog record-level page (more likely).

[I'm actually very open to just plain leaving it out.]

The format is a hash keyed on the HathiTrust record ID, which can currently be turned into a URL such as [http://catalog.hathitrust.org/Record/003384758](http://catalog.hathitrust.org/Record/003384758). Elements are:

  * *recordURL*: The URL to the human-readable record view in the catalog
  * *titles*: an array of all the full 245, space-separated subfields. Always present, usually with one item, sometimes more than one (vernacular entries), almost never with zero.
  * *isbns*: An array of all the ISBNs associated with the record. Always present; an empty array if none.
  * *issns*, *oclcs*, *lccns*: Same as ISBNs, but for the appropriate data.

Note that at this time, LCCNs are taken from the 010, so the LCCN array will either be empty or have one item. I left it as an array just for consistency.

##### items

This is an array of items, taken from *all* the matched records and ordered (as best I can) based on their enumcron. If no enumcron is present, order is undefined.

  * *fromRecord*: The HathiTrust record ID, as used as a key in the hash of records (explained above).
  * *htid*: The HathiTrust ID for the item.
  * *itemURL*: The URL to the page-turner (or search box, for search-only items) for this item. It's currently just appended to the prefix "http://hdl.handle.net/2027/", but I thought I'd include it in case the preferred URL algorithm changes at some point.
  * *rights*: The rights code for this item, as explained at [http://www.hathitrust.org/hathifiles_metadata](http://www.hathitrust.org/hathifiles_metadata).
  * *orig*: The institution that supplied the item for digitization.
  * *lastUpdate*: The date of the last time this item or its containing record was touched, either because of  ingest by the HathiTrust system or later editing, as YYYYMMDD. May be 00000000 if unknown.
  * *enumcron*: (OPTIONAL) The enumeration/cronology (e.g., "v. 3 1997" or somesuch). Again -- optional. Leave out the key? Provide an empty string? Provide a *false*?

##### A word about enumcron

The enumcron string is fickle, and very local. The algorithm I'm using to sort them basically consists of taking all the numbers in the enumcron strings and zero-padding them to 8 digits, then sorting. It works pretty well, but isn't perfect. I'm incredibly resistant to trying to do anything fancier, simply because I want it to be fast and because trying to deal with all possible enumcron formats is a sisyphean task.  

#### An actual record

Here's the simplest possible case: a single matched record with a single item


~~~javascript

  {
    "records":
      {
        "000366004":
          {
            "recordURL" : "http://catalog.hathitrust.org/Record/000366004",
            "titles": ["The Sneetches, and other stories. Written and illustrated by Dr. Seuss."],
            "isbns": [],
            "issns": [],
            "oclcs": ["00470409"],
            "lccns": ["68001537"]
          }
      }
    "items": [
      {
        "fromRecord": "000366004",
        "htid": "mdp.39015079651611",
        "itemURL": "http://hdl.handle.net/2027/mdp.39015079651611",
        "rightscode": "ic",
        "lastUpdate": "20091004",
        "orig": "University of Michigan",
        "enumcron": false
      }
    ],
  }

~~~

We can see that (despite my expectation) we don't happen to have an ISBN for this item. The item originally came from Michigan, either ingested or last updated on October 10th, 2009. The HathiTrust catalog page for this item is [http://catalog.hathitrust.org/Record/000366004](http://catalog.hathitrust.org/Record/000366004) (derived from the record ID) and it is In Copyright (ic), so the itemURL goes to a page that allows only search.

### Making the request

I'll take care of normalizing data on the way in (mostly done by the Solr backend): strip leading zeros off the OCLC number, normalize the LCCN as per [this page at the Library of Congress](http://www.loc.gov/marc/lccn-namespace.html#syntax), strip anything funny-looking from the ISBN and ISSN, and (probably) convert all ISBNs into ISBN13s.

I'm anticipating three formats for a request (note: they don't work yet. There's no code):

#### Single-identifier option

    http://catalog.hathitrust.org/api/volumes/oclc/00470409.json
    http://catalog.hathitrust.org/api/volumes/lccn/68001537.json
    http://catalog.hathitrust.org/api/volumes/issn/1051290x.json
    http://catalog.hathitrust.org/api/volumes/isbn/0835221792.json  

Simple and unambiguous; returns the proposed return structure as described above (and presumable amended before actual implementation). Again, any normalization that needs to be done will be done on my end, so "00470409" and "470409" are considered the same OCLC number.

#### Multiple-identifier, multi-request option

    http://catalog.hathitrust.org/api/volumes?yourID1=oclc:00470409|lccn:68001537&amp;yourID2=oclc:67890987|isbn:987652348X

In this format, you can see that (a) you can provide multiple pieces of metadata for a record, separated by pipe characters (|),  and (b) you can provide metadata sets for multiple records at once, keyed on whatever arbitrary ID you want to use.

The return format would look like this:


~~~javascript

  {
    "yourID1" : <proposed return structure>,
    "yourID2" : <proposed return structure>,
      ...
  }

~~~

#### What to do when the provided metadata don't agree?

It's entirely possible to provide an OCLC number and an LCCN that, in fact, refer to two different records. It's also possible that we have two records in the system that should be merged, but haven't been.

Some possible algorithms:

  1. *Require that all sent numbers match*: If you send an OCLC, and ISBN, and an LCCN, any returned record must have all three, and all must match. That seems too strict.
  2. *Return any records that match any sent numbers*: I could do a boolean-OR, so any record that matches any of the numbers you send gets returned. The risk of returning too much data seems too great.
  3. *Return any records that don't mismatch any sent numbers*: The same as the first option, but null matches anything. So, *if* you sent an LCCN, and *if* the record has an LCCN, they must match. *If* you sent an OCLC number and *if* the record has an OCLC number, it, too, must match, etc.. Basically, every piece of metadata, if provided, must match.
  4. *Order the number types and only match the best available*. We provide an ordered list of type: OCLC, LCCN, ISBN, and finally ISSN. If you provide an OCLC number and there is a record with that OCLC number, return it and ignore everything else. If you didn't provide an OCLC number (or if you did but we didn't get any matches), move on to the LCCN and try again, as shown below.


~~~

    // The algorithm for #4
    foreach type in (OCLC, LCCN, ISBN, ISSN) {
      next unless (providedSearch[type]); ## move on unless a number was provided
      records = recordsThatMatch(type, providedSearch[type]);
      if records.size > 0 { # If we found some, return
        return records;
      }
      ## else, we move to the next type.
    }

~~~~


So, for #4, if you provide an OCLC number and we find a match or matches, stop looking and return them. If we don't find an OCLC match but you also provided an LCCN, look for records that match the LCCN, and if found return *them*. Repeat with ISBN and ISSN.

#### Understanding #3 vs. #4

Suppose the following are true:

  * You provide an OCLC number *O* and an LCCN *L*
  * I have a record r1 with OCLC number *O* and no LCCN at all
  * I have a record r2 with LCCN *L* and no OCLC number at all.

Under option #3, both records would be returned. They both fulfill the criteria that they match all the supplied identifiers in all fields for which they have values. In other words, *r1* has a positive match on OCLC (*O* == *O*) and a null-matches-everything match on LCCN (*L* == no data).

Under option #4, only *r1* is returned. We first look for all records that match on the OCLC number provided, find exactly one, and return it. We never even bother to look for records that match on LCCN only.

#### Let's pick one and see how it works in the real world

I'm leaning toward #4, but I'm open to #3 as well, or any other variant that can be computed quickly and easily on this end. We're talking about some pretty weird edge cases when we start going down this road, and I don't want to sacrifice ease of use and ease of computation any more than we have to.

### Please comment!

You can comment here, or send email directly to me. I'll follow up this post periodically with more thoughts and synopses of what I've heard.
