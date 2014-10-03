---
title: "Beta version of the HathiTrust Volumes API available"
date: 2009-12-15
layout: post
slug: beta-version-of-the-hathitrust-volumes-api-available

---

## <span style="color: red">MAJOR CHANGE</span>

So, initially, this post listed that the way to separate multiple simultaneous requests was with a nice, URL-like slash (/) character.

Then, I remembered that LCCNs can have embedded slashes, e.g., 65063380//r85.

So, we're back to using pipe (\|) characters to separate multiple calls -- the examples below have been updated to reflect this.


### Introduction

I've put up a beta version of the HathiTrust Volumes API [previously discussed](http://robotlibrarian.billdueber.com/thinking-through-a-simple-api-for-hathitrust-item-metadata/) on this blog and via email.

Currently, I've only got json output, although there is space in there for other output formats as necessary.

### What exactly is this?

Given: an identifier or set of identifiers, this API will
Return: a set of matched records and a sorted list of the items available in the HathiTrust.

Useful, for example, if you want to display HathiTrust holdings alongside your own in your OPAC.


### Simple, single-value call

Given the URL:

[http://catalog.hathitrust.org/api/volumes/oclc/15420548.json](http://catalog.hathitrust.org/api/volumes/oclc/15420548.json)

You'll get the following back:


~~~javascript

  {
      "records":
      {
          "000791709":
          {
              "recordURL":"http://catalog.hathitrust.org/Record/000791709",
              "titles":
              [
                  "\"Zhong gong dang shi\" fu dao /",
                  "\u300a\u4e2d\u5171\u515a\u53f2\u300b\u8f85\u5bfc /"
              ],
              "isbns": [],
              "issns": [],
              "oclcs": ["15420548"],
              "lccns": []
          }
      },
      "items":
      [
          {
              "orig":"University of Michigan",
              "fromRecord":"000791709",
              "htid":"mdp.39015058510069",
              "itemURL":"http://hdl.handle.net/2027/mdp.39015058510069",
              "rightsCode":"ic",
              "lastUpdate":"00000000",
              "enumcron":false
          }
      ]
  }  

~~~

Note that the 'records' are keyed on the local umid, also available in the 'fromRecord' field of each item.

The generic short form is:

    http://catalog.hathitrust.org/api/volumes/(idtype)/id.(outputtype)

Right now the valid idtypes are:

  * issn (will be normalized to just digits, no leading zeros)
  * isbn (will be normalized to an ISBN-13)
  * oclc (will be normalized to all digits, no leading zeros)
  * lccn (will be normalized [as recommended](http://www.loc.gov/marc/lccn-namespace.html#syntax))
  * htid (HathiTrust item id, seen above as "mdp.39015058510069")
  * umid (the University of Michigan record ID, seen above in the "fromRecord" field of an item)

Currently the only valid outputtype is 'json'.


### More complex, multi-valued call

The full API URL is [this]

[http://catalog.hathitrust.org/api/volumes/json/id:1;oclc:45678;lccn:70628581\|id:2;isbn:1591581613](http://catalog.hathitrust.org/api/volumes/json/id:1;oclc:45678;lccn:70628581\|id:2;isbn:1591581613)

This is a request for data on two separate items, identified on the calling end as simply '1' and '2' (id:1 and id:2). The first item is searched for using both an oclc number and an lccn; the second supplies only an isbn.

Note that

  * The output format (json) has moved to appear right after the '/volumes/'
  * There's an arbitrary 'id' field. This will be used to index the return values, so use something meaningful on your end.
  * keys and values are separated by colons. Key-Value pairs are separated by semi-colons.
  * Separate requests are separated by '/' in the URL, allowing you to request data for an arbitrary number of items with a single call.
  * Return values are
  * Matches follow the "#3" option on [the old post](http://robotlibrarian.billdueber.com/thinking-through-a-simple-api-for-hathitrust-item-metadata/), the
   "Must match if present" option -- basically, if you supply an identifier and a record has one of those identifiers, they must match.

So, in the example, the first request has both an oclc number and an lccn. Matches are as follows:

  * If a record has an oclc number but no lccn, its oclc number must match the passed oclc number.
  * If a record has an lccn but no oclc number, its lccn must match the passed lccn value.
  * If a record has both an lccn and an oclc number, both its identifiers must match the passed values.

The returned structure is keyed on the arbitrary id passed in the search string (if not present, the whole search string will be used instead):


~~~javascript

  {
      "1":
      {
          "records":
          {
              "001474331":
              {
                  "recordURL":"http://catalog.hathitrust.org/Record/001474331",
                  "titles":
                  ["Some aspects of seventeenth-century medicine &amp; science; papers read at a Clark Library seminar, October 12, 1968"],
                  "isbns": [],
                  "issns": [],
                  "oclcs": ["00045678"],
                  "lccns": ["70628581 //r86"]
              }
          },
          "items":
          [{
                  "orig":"University of Michigan",
                  "fromRecord":"001474331",
                  "htid":"mdp.39015004074095",
                  "itemURL":"http://hdl.handle.net/2027/mdp.39015004074095",
                  "rightsCode":"ic",
                  "lastUpdate":"20090713",
                  "enumcron":false
              }]
      },
      "2":
      {
          "records":
          {
              "004370624":
              {
                  "recordURL":"http://catalog.hathitrust.org/Record/004370624",
                  "titles":
                  ["ARBA in-depth. Philosophy and religion /"],
                  "isbns":
                  ["1591581613"],
                  "issns": [],
                  "oclcs": ["53462174"],
                  "lccns": ["2003065945"]
              }
          },
          "items":
          [{
                  "orig":"University of Michigan",
                  "fromRecord":"004370624",
                  "htid":"mdp.39015058261911",
                  "itemURL":"http://hdl.handle.net/2027/mdp.39015058261911",
                  "rightsCode":"ic",
                  "lastUpdate":"20090907",
                  "enumcron":false
           }]
      }
  }

~~~


### Enumeration / Chronology

An effort is made to return items in "enumcron order" -- hopefully, with earlier volumes showing up before later volumes.

The full enumcron is listed in the items if you need to try something different.

### JSONP Support

[JSONP](http://niryariv.wordpress.com/2009/05/05/jsonp-quickly/) output is supported -- just throw a '&amp;callback=blahblahblah' on the end of the URL you call and you'll get a function definition back.

Some examples:

[http://catalog.hathitrust.org/api/volumes/oclc/15420548.json&amp;callback=myfunc](http://catalog.hathitrust.org/api/volumes/oclc/15420548.json&amp;callback=myfunc)


[http://catalog.hathitrust.org/api/volumes/json/id:1;oclc:45678;lccn:70628581\|id:2;isbn:1591581613&amp;callback=myfunc](http://catalog.hathitrust.org/api/volumes/json/id:1;oclc:45678;lccn:70628581\|id:2;isbn:1591581613&amp;callback=myfunc)
