---
title: "Simple Ruby gem for dealing with ISBN/ISSN/LCCN"
date: 2010-09-13
---

I needed some code to deal with ISBN10->ISBN13 conversion, so I put in a few other functions and wrapped it all up in a [gem called library_stdnums](http://rubygems.org/gems/library_stdnums).

It's only 100 lines of code or so and some specs, but I put it out there in case others want to use it or add to it. Pull requests at the [github repo](http://github.com/billdueber/library_stdnums) are welcome.


Functionality is all as module functions, as follows:

### ISBN

* char = StdNum::ISBN.checkdigit(ten-or-thirteen-digit-isbn)
* boolean = StdNum::ISBN.valid?(ten-or-thirteen-digit-isbn)
* thirteenDigitISBN = StdNum::ISBN.convert_to_13(ten-or-thirteen-digit-isbn)
* tenDigitISBN = StdNum::ISBN.convert_to_10(ten-or-thirteen-digit-isbn)

### ISSN

* char = StdNum::ISSN.checkdigit(issn)
* boolean = StdNum::ISSN.valid?(issn)

### LCCN

* normalizedLCCN = StdNum::LCCN.normalize(lccn)

Again, there's nothing special here -- just letting folks know it's out there.
