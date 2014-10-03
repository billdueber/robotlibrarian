---
title: "Ruby gem library_stdnums goes to version 1.0"
date: 2011-05-06
tags: "librarian code, ruby"
---

I just released another (this time pretty good) version of my gem for normalizing/validating library standard numbers, `library_stdnums` ([github source](https://github.com/billdueber/library_stdnums) / [docs](http://rubydoc.info/github/billdueber/library_stdnums/master/frames)).

The short version of the functions available:

* **ISBN**: get checkdigit, validate, convert isbn10 to/from isbn13, normalize (to 13-digit)
* **ISSN**: get checkdigit, validate, normalize
* **LCCN**: validate, normalize

Validation of LCCNs doesn't involve a checkdigit; I basically just normalize whatever is sent in and then see if the result is syntactically valid.

My plan in my Copious Free Time is to do a Java version of these as well and then stick them into a new-style Solr v.3 filter so I (and, by extension, you, if you're interested) can have Solr do normalization during both index and search time.
