---
title: "An exercise in Solr and DataImportHandler: HathiTrust data"
date: 2009-09-28
---

Many of the folks who read this blog (hi, both of you! Mom, say hello to Dad!) are aware, at least tangentially, of the [HathiTrust](http://www.hathitrust.org/). Currently hosted by us at the [University of Michigan](http://www.lib.umich.edu/), the most public interface to its data is a [VuFind](http://www.vufind.org/) installation you can access at [catalog.hathitrust.org](http://catalog.hathitrust.org) (or, for you smart-phone types, at [m.catalog.hathitrust.org](http://m.catalog.hathitrust.org/)). Once you do a metadata search, you get links into the actual page images or a chance to search the fulltext of the selected item (depending on its copyright status).

It's awesome. Seriously. Even in the absence of fulltext, being able to search within an item can be incredibly useful. Give it a shot if you haven't. 

### You don't always need an OPAC

But there are plenty of folks who don't want or need a full-flown interface into all the metadata. They've already got one of those. What they're interested in, mostly, is figuring out how to easily put links in their *own* OPAC (or whatnot or whoseits) to the HathiTrust if page images or searching are available. See, for example, [a typical record from Tod Olson's stuff at U-Chicago](http://lens.lib.uchicago.edu/?itemid=library/marc/uc\|835086) -- he sniffs for HathiTrust and Google Books availability via embedded javascript.

To this end, the HathiTrust folks provide a set of [simple, tab-delimited files](http://www.hathitrust.org/hathifiles) -- a full extract on the first of every month, and nightly updates every ...er...night. 

You can see from the [description of the file](http://www.hathitrust.org/hathifiles_metadata/) that it's very simple. Tab-delimited fields of the HathiTrust ID, right information, and all the golden-oldie standard identifiers -- some of which (ISSNs, ISBNs, etc.) are further comma-delimited in cases where multiple values are available and a field repeats. And a title and *enumcron* (description of an individual volume, e.g., "Sept 2007, vol. 33, issue 4"), so you have something useful to display if you need to, and that's 98% of what most folks want.

### The smart way to do it: RDBMS

If you want to query this data quickly and easily, the obvious thing to do is to dump it into a database. One main table for the non-repeated values, and either a few key=>value tables (or, if you're lazy, a single key => type/value) for the repeated ISBNs/ISSNs/whatnot. A quick mod-perl script to set up some data normalization going in and out and persist the prepared SQL queries and you're set. 

It's hard to make an argument *against* using a database for these data. I mean, c'mon. We've got a well-defined structure. An obvious foreign-key. No full-text searching needed. This is practically *designed* for a good old-fashioned RDBMS. Plus, I've done this approximately a zillion times before, so I'm good and fast at it. Case closed.

### How I'm gonna do it

Screw that. What I really wanted to do was start messing around with the [DataImportHandler](http://wiki.apache.org/solr/DataImportHandler)(DIH) in Solr. 

I can make a weak argument for including the data in a Solr instance. To wit, it'll certainly be fast enough for anything I'm gonna throw at it, and (more important to me) it's easy to set up datastore-level indexing and querying filters with [built-in facilities](http://robotlibrarian.billdueber.com/easy-solr-types-for-library-data/) and/or [custom code](http://robotlibrarian.billdueber.com/building-a-solr-text-filter-for-normalizing-data/). This allows me to build clients that call it without having to worry about manipulating the input much, if at all.

The list of simple DIH examples is...well, I never really found any good ones, although I'm sure they're out there. The documentation isn't bad, but it's not full of complete examples, and almost all of them have to do with the potential complexities of sucking data out of a database, which is what most people want to do. Not me, I've got flat files to work with.

Luckily, you can fire up an "interactive" DIH session where, at the very least, you can try to import a few rows of data and see if things are puking. I didn't find the error reports particularly helpful all the time, but it's about a zillion times better than nothing, I can tell you that much.

### The game plan

We'll start with the assumption that I've already managed to load a full dump from some date (run with me here; I'll explain how to do it later). Then what we want to do is the following:

 1. Every night, download the nightly additions/changes file and gunzip it. 
 2. Hit the DIH handle to import all files that (a) have a filename of the right format, and (b) have a created date after the last time the DIH handle was run.
 
And that's it. Get the new stuff, have DIH figure out what's new, and import it. 

The first part is easy enough to do with perl/python/ruby/whatever. I'll leave it as an exercise for all you diligent students. 


#### Setting up solrconfig.xml

This is the easy part. Set up the handler, give it a semi-meaningful name, and 
call out to a config file.


~~~xml

  <requestHandler name="/hathiimport" class="org.apache.solr.handler.dataimport.DataImportHandler">
      <lst name="defaults">
        <str name="config">hathi-data-config.xml</str>
      </lst>
  </requestHandler>  

~~~

#### Define some useful data types in `schema.xml`

I left pretty much all of the boilerplate in `schema.xml` and just added a few types to deal with identifiers. 

  * *lowercase*: return a single token that's been lowercased. Don't muck with it otherwise.
  * *genericID*: trim it, lowercase it, ditch everything that's not a number or a letter, and return as a single token.
  * *numeric*: Ditch everything but the first string of digits, and *then* ditch any leading zeros. Useful when you know it's gotta be an integer.
  * *stdnum* Find the first set of digits (optionally followed by an 'X' and potentially interspersed with dashes or dots), strip off the leading zeros,  and return it. Good to extract an ISBN from a string like "(alt) 123-45-678X electronic only".
  * *lccnnormalizer*: Custom code to normalize an LCCN as per [this page at the LoC]( http://www.loc.gov/marc/lccn-namespace.html#syntax).
  


~~~xml

<types>
  <!-- lowercases the entire field value, keeping it as a single token.  -->
<fieldType name="lowercase" class="solr.TextField" positionIncrementGap="100">
  <analyzer>
    <tokenizer class="solr.KeywordTokenizerFactory"/>
    <filter class="solr.LowerCaseFilterFactory" />
  </analyzer>
</fieldType>

<!-- Full string, stripped of \W and lowercased -->
 <fieldType name="genericID" class="solr.TextField" sortMissingLast="true"  omitNorms="true">
   <analyzer>
     <tokenizer class="solr.KeywordTokenizerFactory"/>
     <filter class="solr.LowerCaseFilterFactory"/>
     <filter class="solr.TrimFilterFactory"/>
     <filter class="solr.PatternReplaceFilterFactory"
          pattern="[^\p{L}\p{N}]" replacement=""  replace="all"
     />
   </analyzer>
</fieldType>

  <!-- standard number normalizer - extract sequence of digits, strip leading zeroes -->
<fieldType name="numeric" class="solr.TextField" sortMissingLast="true" omitNorms="true" >
 <analyzer>
   <tokenizer class="solr.KeywordTokenizerFactory"/> 
   <filter class="solr.LowerCaseFilterFactory"/>
   <filter class="solr.TrimFilterFactory"/>
   <filter class="solr.PatternReplaceFilterFactory"
        pattern="[^0-9]*([0-9]+)[^0-9]*" replacement="$1"
   />
   <filter class="solr.PatternReplaceFilterFactory"
        pattern="^0*(.*)" replacement="$1"
   />
 </analyzer>
</fieldType>


  <!-- Simple type to normalize isbn/issn. Just get first string of digits followed by an optional 'x' -->
<fieldType name="stdnum" class="solr.TextField" sortMissingLast="true" omitNorms="true" >
 <analyzer>
   <tokenizer class="solr.KeywordTokenizerFactory"/>
   <filter class="solr.LowerCaseFilterFactory"/>
   <filter class="solr.TrimFilterFactory"/>
    <filter class="solr.PatternReplaceFilterFactory"
        pattern="^[\s0\-\.]*([\d\.\-]+x?).*$" replacement="$1"
   />
   <filter class="solr.PatternReplaceFilterFactory"
        pattern="[\-\.]" replacement=""  replace="all"
   />
 </analyzer>
</fieldType>

<!-- LCCN normalization on both index and query -->
<fieldType name="lccnnormalizer" class="solr.TextField"  omitNorms="true">
  <analyzer>
    <tokenizer class="solr.KeywordTokenizerFactory"/> 
    <filter class="solr.LowerCaseFilterFactory"/>
    <filter class="solr.TrimFilterFactory"/>
    <filter class="edu.umich.lib.solr.analysis.LCCNNormalizerFilterFactory"/> 
  </analyzer>
</fieldType>

<!-- since fields of this type are by default not stored or indexed,
     any data added to them will be ignored outright.  --> 
<fieldtype name="ignored" stored="false" indexed="false" multiValued="true" class="solr.StrField" /> 
   
</types>

~~~

#### Add field definitions to `schema.xml`

This is pretty straight-forward: just set it up.


~~~xml

<field name="htid"        type="genericID"          indexed="true"  stored="true"  multiValued="true"/>
<field name="bibnum"       type="genericID"         indexed="true"  stored="true"/>

<field name="access"       type="lowercase"         indexed="true"  stored="true"/>

<field name="rights"       type="lowercase"         indexed="true"  stored="true"/>

<field name="source"       type="lowercase"         indexed="true"  stored="true"/>
<field name="sourceid"     type="genericID"         indexed="true"  stored="true"/>

<field name="lccn"         type="lccnnormalizer" indexed="true"  stored="true"  multiValued="true"/> 
<field name="oclc"         type="numeric"        indexed="true"  stored="true"  multiValued="true"/>
<field name="isbn"         type="stdnum"         indexed="true"  stored="true"  multiValued="true"/>
<field name="issn"         type="stdnum"         indexed="true"  stored="true"  multiValued="true"/>

<field name="title"        type="text"         indexed="true" stored="true"/>
<field name="imprint"      type="text"         indexed="true" stored="true"/>
<field name="enumcron"     type="text"         indexed="true" stored="true"/>

  <!-- Ignore the multivalued, comma-delimieted source strings -->

  <field name="rawLine"  type="ignored" indexed="false" stored="false"/>
  <field name="issns"  type="ignored" indexed="false" stored="false"/>
  <field name="isbns"  type="ignored" indexed="false" stored="false"/>
  <field name="oclcs"  type="ignored" indexed="false" stored="false"/>
  <field name="lccns"  type="ignored" indexed="false" stored="false"/>

~~~

#### hathi-data-config.xml -- define how DIH is going to work.

This, of course, is the meat of the heart of the center of the matter. 

I'm going to make use of four DIH technologies:

  * **FileDataSource**: In DIH, you declare a data source from which you'll be sucking the raw data for manipulation and massaging. I'm just using a file, so this is for me. You can, as you might expect, pull in from a URL or (as mentioned) a database via JDBC.
  * **FileListEntityProcessor**: Given a directory and a set of criteria for a file, this will return a list of filenames that match those criteria. The criteria we'll be using are (a) a regexp the filename must match, and (b) a creation date after the last time we ran the process.
  * **LineEntityProcessor**: Once you've got a data source, you need to stream it in somehow. There are Processors for XML and other formats, but this one just pulls in lines one at a time. The documentation all talks about `LineEntityProcessor` basically only being useful for pulling in, say, a list of filenames, but since my data is all line-by-line, this is what I'm using as my primary record-fetcher. It populates a single field called `rawLine` for later processing.
  * **RegexTransformer**: Allows you to take a field pulled from the datasource (or already derived from previous processing) and do regexp substitutions, group extraction, or splitting. 

SO...I'm going to:

  1. Set up a `FileDataSource` to read from files
  2. Use `FileListEntityProcessor` to get a list of files that match my criteria
  3. Run each through `LineEntityProcessor` to generate a bunch of `rawLine`s.
  4. Use the `RegexTransformer` multiple times to extract the data from the line.
  
[If you never went to look at it, this might be a good time to check out the [description of the tab-delimited metadata files](http://www.hathitrust.org/hathifiles_metadata/).]


~~~xml

  <dataConfig>
    <dataSource name="fds" encoding="UTF-8"  type="FileDataSource" />
    <document>
      <!-- Get a list of files from the last time the handler ran -->
      <entity name="hathifile"
              processor="FileListEntityProcessor"
              newerThan="${dataimporter.last_index_time}"
              fileName="^hathi_upd_.*\.txt$"
              rootEntity="false"
              baseDir="/Users/dueberb/Documents/devel/hathi"
      >

        <entity name="hathiline"
                processor="LineEntityProcessor"
                url="${hathifile.fileAbsolutePath}"
                rootEntity="true"
                dataSource="fds"
                transformer="RegexTransformer"
        >

<!-- Big ugly regexp to get all the tab-delimited fields -->
          <field column="rawLine"
                 regex="^(.*)\t(.*)\t(.*)\t(.*)\t(.*)\t(.*)\t(.*)\t(.*)\t(.*)\t(.*)\t(.*)\t(.*)\t(.*)$"
                 groupNames="htid,access,rights,bibnum,enumcron,source,sourceid,oclcs,isbns,issns,lccns,title,imprint"
          />

<!-- Split the multi-values on comma -->

          <field column="oclc" splitBy="," sourceColName="oclcs" />
          <field column="issn" splitBy="," sourceColName="issns" />
          <field column="isbn" splitBy="," sourceColName="isbns" />
          <field column="lccn" splitBy="," sourceColName="lccns" />
        </entity> <!-- end of hathiline -->

      </entity> <!-- end of hathifile --> 
    </document>
  </dataConfig>  

~~~

### And...it doesn't work.

It *almost* works. The problem is that my attempt to use the variable `${dataimporter.last_index_time}` is busted. There's [a ticket to fix it](https://issues.apache.org/jira/browse/SOLR-1473) and a patch already provided, so it's only a matter of time before it's not an issue. 

For the moment, though, we'll change that line to:


~~~xml

  <entity name="hathifile"
          processor="FileListEntityProcessor"
          newerThan="'NOW/DAY'"
          fileName="^hathi_upd_.*\.txt$"
          rootEntity="false"
          baseDir="/Users/dueberb/Documents/devel/hathi"
  >

~~~

That says to basically take everything created since midnight and use it. If you have cron scripts set up to run this every day, you'll have no problems.

### Dealing with a full extract

You'll only have to do this once, of course, but it has to be done. Basically, reproduce the DIH handler with a different name, pulling in the data from a full extract (you could, e.g., just change the *filename* parameter to accept `/^hathi_full_.*\.txt$/`). Maybe call it *hathifullimport* instead of *hathiimport*.

### Fire her up!

Once you're ready to go, just hit the right URL:

    http:://solrmachine:port/solr/hathifullimport?command=full-import&clean=true
    
    http:://solrmachine:port/solr/hathiimport?command=full-import&clean=false
    
The first one will get the initial big, full file; the second will pull in all the nightlies you've downloaded, gunzipped, and put in the right place (provided, of course, they're dated after the last midnight, or they've fixed DIH to allow the *last_index_time* syntax).

### Next steps?

Beer or wine. Take your pick. 

After that, though, it'd be a matter of actually writing the download scripts and setting up cron jobs. And, of course, putting a front-end on it if you want, or massaging the data as they come out to return a nice JSON format for your consumers. That sort of thing.

### So, wait...is this really worth doing?

Maybe. Probably not. It was worth it to me to start thinking about DIH and how I can use it. And it might be worth it to you, if you want to play around with these data in the ways that solr makes easy. 

But, like to many things, it's less worth *doing* that it was worth *writing up*. I learned a lot.
