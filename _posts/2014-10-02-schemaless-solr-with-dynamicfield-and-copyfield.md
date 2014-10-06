---
title: "\"Schemaless\" solr with dynamicField and copyField"
tags: solr
date: 2014-10-02
layout: post
slug: schemaless-solr-with-dynamicfield-and-copyfield
---

[Holy Kamoly, it's been a long time since I blogged!]

Recent versions of [solr](https://lucene.apache.org/solr/) have the option to run in what they call ["schemaless mode"](https://cwiki.apache.org/confluence/display/solr/Schemaless+Mode), wherein fields that aren't recognized are actually added, automatically, to the schema as real named fields.

I find this intruguing, but it's not what I'm after right now.

The problem I'm in the first stages of addressing is that my `schema.xml` is huge mess -- very little consistency, no naming conventions dictating what's stored/indexed, etc. It grew "ogranically" (which is what I say when I mean I've been lazy and sloppy) and needs a full-on reorganization.

The way people tend to address this is with strict naming conventions (possibly using [dynamicField](https://cwiki.apache.org/confluence/display/solr/Dynamic+Fields) ) and judicious use of [copyField](https://cwiki.apache.org/confluence/display/solr/Copying+Fields) directives. The [Project Hydra](http://projecthydra.org/) folks have [a nice, straightforward system](https://github.com/projecthydra/hydra/wiki/Solr-Schema) for how they set up dynamic fields.


## Indexed XOR Stored?

The more I thought about it, the more I wondered whether it might be useful to have a *strict separation of stored and indexed fields*. Indexed fields would be named with an appropriate suffix, so you know how they've been analyzed. And stored fields would have pleasant, human-readable names to make them easy to deal with for consuming applications.

What I *think* I'd like is a system where:

* All stored fields have 'bare' names (e.g., 'title', not 'title_t' or 'title_s')
* All indexed fields are typed according to their name (so I know 'title_t' is an indexed field of type "text")
* Separation of stored and indexed fields -- a field is either stored or indexed, but not both.
* A "schemaless" setup, where I don't need to define all (any of?) my fields in my schema and reboot solr when I make a change.


To be clear: I'm not sure this is a great way to go as of yet. But I figured out what I think is a good way to do it, should it turn out to be worthwhile.

## Part 1: Dynamic Fields

Solr allows one to define dynamic fields -- a field whose type is determined by a glob-match on its name. Instead of explicitly naming your field in your schema, you can do something like:

~~~xml
<dynamicField name="*_is" type="int" indexed="true" stored="true" multiValued="true"/>
~~~

...to indicate that any unrecognized field whose name ends in `_is` will treated as an indexed, stored integer.

Dynamic Field definitions are processed in order of declaration; first one wins. That allows you to define a "default" as the very last `dynamicField` that matches *anything* (e.g., `*`). The `schema.xml` that ships with Solr suggests that you can use this functionality to just ignore unrecognized fields.

~~~xml
<dynamicField name="*" type="ignored" multiValued="true" />
~~~

But that gives me an idea.

## Part 2: Copy Fields

The `copyField` directive allows you to index the same text into multiple fields (presumably with different analysis chains). Index data into one field, it automatically gets copied into another.

~~~xml

<field name="title" type="text" indexed="true" stored="true" />
<field name="title_l" type="text_leftanchored" indexed="true" stored="false"/>

<copyField source="title" dest="title_l"/>

~~~

In this case, even though I only send a `title`, the indexed field `title_l` will automatically be created and available for me to search against. Nice.

## Part 3: Copy Field with globs

But it gets better. You can have globs (`*`) in your copyField source or destination attributes.

~~~xml
<!-- Copy all text fields (those that end in '_t') into 'keywords' -->
<copyField source="*_t" dest="keywords"/>
~~~

So that's nice. But what if you have globs in both the source and the destination? The docs say:

> The copyField command can use a wildcard (*) character in the dest parameter only if the source parameter contains one as well. copyField uses the matching glob from the source field for the dest field name into which the source content is copied.

Hmmmmmm....

## Part 4: Putting it all together

Once I read that, I thought, "Huh. I'm hungry."

But after lunch, I thought, "Maybe I can do something with this."

Here's what I came up with.

~~~xml
<dynamicField name="*_t_s" type="text" indexed="false" stored="false" multiValued="true"/>
<dynamicField name="*_t" type="text" indexed="true" stored="false"/>

<copyField source="*_t_s" dest="*_t"/>
<copyField source="*_t_s" dest="*"/>

<!-- The default: a multivalued, stored, non-indexed string -->
<dynamicField name="*" type="string" indexed="false" stored="true" multiValued="true"/>
~~~

Let's walk through that.

First, there are two `dynamicField` definitions. The first is a no-op: unstored, unindexed. We use it only for copying. The second is a standard indexed (but not stored) text field.

Then come the `copyField`s, where we match on the suffixes of the field types.
Finally, we have our default: a stored, unindexed string. (Note that when Solr stores a value, it stores whatever you put into it, not the value after analysis -- same as a String does anyway).

Suppose I index an undeclared field called `title_t_s`:

* `title_t_s` matches the first `dynamicField` declaration. This specific field is ignored (no indexing, no storing), but the text sent to it remains available for further processing by the `copyField`s.
* The first `copyField` matches, and copies the text into newly-generated field formed by what matched the `*` in the source field, followed by `_t`. That's `title`, so we get `title_t`.
* The newly-minted `title_t` field is also unrecognized, but it matches the *second* `dynamicField` and is thus assigned to be an indexed text field.
* Meanwhile, the second `copyField` *also* matches our original `title_t_s`. It uses what matched against the `*` in the source (`title`, again) to create a new field just called `title`.
* Now we have a new field called `title` not matching any declared field, so it runs down the list of `dynamicField` definitions until it hits our stopgap at the end: a stored, nonindexed string.

Yeah, like that wasn't confusing. 

The result is what's important, though. What we end up with field-wise is:

* `title_t_s` disappearing into the ether. It's just gone.
* `title_t`, an indexed text field
* `title`, a stored string.

Now I can run searches against `title_t`, but my document will have a nice stored string in it just called `title`.

## Why this is probably a bad idea.

Depending on how crazy you want to get options-wise (multi-valued or not, termVectors or not, etc.) you can get a combinatorial explosion on the number of `dynamicField`/`copyField` sets you need to generate. But that's not the real problem.

The real problem is that you don't have *any* intrinsic documentation of what your index looks like. None. You can't even look at the indexing code, because it'll look like you're sending a document with a field called `title_t_s` and that field is nowhere to be found.

So, like I said: interesting, but by no means the obvious way to go. Still, I'm sure I'll have some variant of this in my schema when it comes time for me to reboot the library catalog.
