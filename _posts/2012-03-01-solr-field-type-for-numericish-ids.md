---
title: "Solr Field Type for numeric(ish) IDs (SST #1)"
date: 2012-03-01
tags: "solr, Stupid Solr Tricks"
layout: post

---

[For the introduction to this series, take a quick gander at [the introduction](http://robotlibrarian.billdueber.com/stupid-solr-tricks-introduction/)]


Like everyone else in the library world, I've got a bunch of well-defined, well-controlled standard identifiers I need to keep track of and allow searching on.

You know, well-vetted stuff like this:

* 1234-5678
* 123-4567-890
* 12-34-567-X
* 0012-0045
* ISBN13: 1234567890123
* ISSN: 1234567X (1998-99)
* ISSN (1998-99): 1234567X
* 1234567890 (hdk. 22 pgs)
* 9
* Behind the 3rd floor desk
* Henry VIII

[Note: some of these may be a titch exaggerated]

How does your system deal with these on index? How about on query?

Here's an idea of how to use a custom solr fieldtype to do the heavy lifting.

### What we're shooting for

I'd like to be able to send in a text string as follows:

* The input can contain other text besides the id
* The ID starts with a digit and consists solely of digits and (optional) dashes, then ends with a digits and possibly a trailing 'X' or 'x' so we can deal with ISBN/ISSN
* The ID has to be at least N characters long (for this example, I'm using N=8); this helps us avoid other text that might trivially look like an ID but isn't.
* Only the ID itself is indexed
* If no valid ID is identified, nothing is indexed


### The numericID field, suitable for ISBN/ISSN/OCLC/etc.

Let's take a look at the end product and then walk through it.


~~~xml
<fieldtype name="numericID" class="solr.TextField"
           positionIncrementGap="1000" omitNorms="true">
<analyzer>
  <tokenizer class="solr.KeywordTokenizerFactory"/>
    <filter class="solr.PatternReplaceFilterFactory"
              pattern="^.*?(\p{N}[\p{N}\-\.]{6,}[xX]?).*$"
              replacement="***$1" />
    <filter class="solr.PatternReplaceFilterFactory"
              pattern="^[^\*].*$" replacement="" />
    <filter class="solr.PatternReplaceFilterFactory"
              pattern="^\*\*\*" replacement="" />
    <filter class="solr.LowerCaseFilterFactory"/>
    <filter class="solr.PatternReplaceFilterFactory"
              pattern="[^\p{N}x]" replacement=""
              replace="all" />
    <filter class="solr.LengthFilterFactory" min="8" max="14" />
    <filter class="solr.PatternReplaceFilterFactory"
              pattern="^0*" replacement=""
    />
  </analyzer>
</fieldtype>
~~~

### Things we'll be learning about today

**NOTE: I really, really recommend taking a look at [Scaling Lucene and Solr](http://www.lucidimagination.com/content/scaling-lucene-and-solr) by the good folks over at [Lucid Imagination](http://www.lucidimagination.com/) for great, short explanations of _omitNorms_, term frequencies, etc.**

Since this is the first post, I'll go over some stuff that's probably a little
too basic for any audience that's likely to show up here, but what the heck.

* KeywordTokenizer
* PatternReplaceFilterFactory
* LowerCaseFilterFactory
* LengthFilterFactory

#### Step 1: "Tokenize" to a single token

The job of a _tokenizer_ is to decide how to split your input into individual tokens (often "words"), which are then munged by any filters you're applying.

For the case of an ID, _we don't want to tokenize_. At least at this juncture, I'm not trying to extract multiple valid IDs out of a single string; I'm just trying to determine if there's a valid ID in there somewhere and throwing everything else away.

In other words, I'm going to treat the input as a _single token_, and then munge the bejeebers out of it in order to get what I want.

In the Solr world, that leads us to the confusingly-named [KeywordTokenizer](http://wiki.apache.org/solr/AnalyzersTokenizersTokenFilters#solr.KeywordTokenizerFactory).

**What we have now**: exactly what we started with

#### Step 2: Find the first thing that looks like an ID and mark it

I primarily work in Ruby and Perl, which means the dramatic abuse of regular expressions is just part of my daily life.

Line 5 is our first use of a regexp in the filter chain via [PatternReplaceFilterFactory](http://wiki.apache.org/solr/AnalyzersTokenizersTokenFilters#solr.PatternReplaceFilterFactory).

The idea is to:

1. Find something that looks like a match
2. If found, get rid of everything else, and throw a '***' onto the beginning so later on I can tell if I matched or not.

The second step is a little...odd...but necessary because I need a way to know if I found a candidate ID or not. If I did, well, there will be three asterisks on the front of the string from here on out. If not, there won't.

This is a little confusing as these things go, so I'll break it down.

Line 6: the match:

* Skip any amount of stuff we don't care about (.*?)
* Match a number (\p{N}) (that's unicode regexp syntax, if you haven't seen it)
* Match a string of at least 6 numbers and dashes
* Close with an optional X or x [Xx]?
* ...and any trailing bits until the end of the string.

So..._[number][six numbers/dashes][optional X]_

At minimum, that's seven digits/dashes.

Line 7: replacement

* Replace the whole string (note how I anchored the match with ^ and $?) with whatever was matched inside the parentheses (represented here by $1) after prepending a set of three asterisks '***'

**What we have now**: If we found a candidate ID, we have that string prepended by '***'. Otherwise, we have exactly what we started with.

#### Step 3: If we didn't find a match, throw it all away

Line 9 shows an attempt to match on any string that start with an asterisk (which we're pretty sure we won't see because that's illegal lucene wildcard syntax). If we have a string that doesn't start with an asterisk, then throw the whole damn thing away because we don't have a candidate ID anyway.

[There's a strong argument to be made that using an asterisk as the tagging character is a bad choice. Anyone have suggestions?]

**What we have now**: Either a candidate ID string preceded by '***' or the empty string.

#### Step 4: Ditch the '***' used to mark a candidate ID

Lines 10-11

Find the '***' and throw it away.

**What we have now**: The raw candidate ID string or the empty string.


#### Step 5: Lowercase it

Line 12.

By 'it' I mean "any X that might be trailing the ID"; we should have thrown everything else away by now. (Note: could have done this with a PattenReplace as well, obviously; not sure why'd I'd choose one over the other).

**What we have now**: The raw candidate ID string with its optional trailing 'X' lowercased, or the empty string

#### Step 6: Get rid of everything that's not a number or an 'x'

Lines 13-15

Ditch any dashes that are remaining. I'm doing it like this instead of just ditching the dashes because I'll likely modify this at some point to allow, e.g., periods between numbers, or maybe spaces. This is safer.

Note the extra parameter (replace="all"), indicating that I want to replace all occurrences. This hasn't been an issue until now because I've been careful to match the entire string by anchoring the pattern at the beginning ('^') and end ('$').

**What we have now**: A string of numbers possibly followed by an 'x', or the empty string.

#### Step 7: Make sure what we have is a reasonable length

Line 16

Now that we've gotten rid of the dashes, we need to make sure we have enough digits left to make a valid identifier.

If we didn't match originally, it quickly got reduced to the empty string, and that will disappear here due to having length 0.

It's also possible that our initial match was, say, '1----3-----6---7', which will at this point have been reduced to just '1367' -- too short for our taste.

In this version, I allow strings of any length between 7 (old OCLC number) and 14 (barcode).

**What we have now**: A string consisting purely of 7-14 characters, the last of which may be an 'x', or nothing at all (e.g., nothing will get indexed).

#### Step 8: Remove leading 0s

My ILS (Aleph) loves to zero-pad all its local identifiers. I'd rather get rid of them.

**What we have now**: What we had before, but with no leading zeros

### Let's try it!

If you're following along at home, get the latest version of the schema and try it!


~~~bash

  cd solr_stupid_tricks
  git pull origin master
  java -jar start.jar

~~~

...and then:

* Go to the analysis page at [http://localhost:8983/solr/admin/analysis.jsp?highlight=on](http://localhost:8983/solr/admin/analysis.jsp?highlight=on)
* Set the first line of the form to use Field: **type** and input _numericID_
* Check the "verbose output" checkbox under _Field value: index_
* Put in a test value and see what the analyzer gives you!

For those of you _not_ following along at home, here are the examples from waaaaaay at the top of this post:

* 1234-5678 => 12345678
* 123-4567-890 => 1234567890
* 12-34-567-X => 1234567x
* 0012-0045 => 120045
* ISBN13: 1234567890123 => 1234567890123
* ISSN: 1234567X (1998-99) => 1234567x
* ISSN (1998-99): 1234567X => **199899**
* 1234567890 (hdk. 22 pgs) => 1234567890
* 9 => [nothing]
* Behind the 3rd floor desk => [nothing]
* Henry VIII => [nothing]

So...not too bad. We did miss one, mistaking a year range for a numeric ID, but if your data are that bad, there's only so much we can do.

### Conclusions

Obviously, this is the tip of the iceberg with this sort of thing. And it can still be confused.

But it _does_ follow our goal of having the exact same behavior on index and query, moving the logic to solr, and being pretty flexible.

Perfect? No. Useful? Yes.
