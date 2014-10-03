---
title: "A plea: use Solr to normalize your data"
date: 2009-03-30
layout: post

---

[Only, of course, if you're using Solr. Otherwise, that'd be dumb.]

We've been working on <a title="Mirlyn2-Beta Library Catalog at the University of Michigan University LIbrary" href="http://mirlyn2-beta.lib.umich.edu/">Mirlyn2-Beta</a>, our installation of VuFind for some time now (don't let the fancy-pants name scare you off), and the further we get into it, the more obvious it is that I want to move as much data normalization into Solr itself as possible.

Arguments about how much business logic to move into the database layer, in the form of foreign-key requirements, cascading inserts and deletes, stored procedures, etc. are as old as the features themselves. Solid arguments for and against are made on all sides, and like all things, there's a happy middle ground for most people. [1. "Most," in this case, excluding the old-time MySQL fanboys who took it as gospel that all data validation and manipulation belongs in the application layer, because their "database" didn't do any of it. Februrary 30th in a date field, anyone?]

But Solr provides an incredibly compelling use case because it allows for data transformation at both index and query time via the use of custom analyzers (or a standard analyzer with text filters applied). We're starting to migrate our schema to use more and more of these things, and I even went so far as to create a custom text filter for LCCNs after being <a href="http://bibwild.wordpress.com/2009/03/11/normalize-your-lccns/">inspired by Jonathan Rochkind.</a>

The incentive is easy to see: client diversity. Let a thousand interfaces bloom, if you can give them all access to the same underlying Solr instance. And, seriously, how many times are you going to write that regexp to semi-normalize ISBNs and ISSNs, huh? Enough already.

If you're using a Solr nightly (and, really, you should be -- faceting is <em>so</em> much faster than the official 1.3 release) you have access to regexp-based filters as well, which makes stuff like this really, really easy:


~~~xml

   <!-- Simple type to normalize isbn/issn/other standard numbers -->
    <fieldType name="stdnum" class="solr.TextField" sortMissingLast="true" omitNorms="true" >
      <analyzer>
        <tokenizer class="solr.KeywordTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.TrimFilterFactory"/>
        <filter class="solr.PatternReplaceFilterFactory"
             pattern="^0*([\d\-\.]+[xX]?).*$" replacement="$1"
        />
        <filter class="solr.PatternReplaceFilterFactory"
             pattern="[\-\.]" replacement=""  replace="all"
        />
      </analyzer>
    </fieldType>


~~~

Here, we use the <em>KeywordTokenizerFactory</em> which, not so intuitively, produces a single token from the input. Then lowercase it and pull of any leading and trailing spaces (Trim).

For those of you that don't read regexp, we then match anything that looks like:

1. Any number of leading zeros
2. ...followed by any number of digits, dashes, or periods and an optional 'X'
3. ...followed by...well, we don't care. Anything else.

...and throw away all but the stuff in #2. Then take <em>that</em> and throw away all the dashes and dots, and you're left with a string of numbers.

The beauty is that it happens both while the index is being made <em>and</em> during query time, so if your user types in " 123-45-6-X  " it will be normalized to 123456x, and then checked against your index.

This is simple stuff, and probably doesn't deserve the virtual ink I'm providing for it, but Vufind out of the box doesn't do any of this sort of thing (likely because "the box" existed before it was super-easy to do this), and we all should be doing it.
