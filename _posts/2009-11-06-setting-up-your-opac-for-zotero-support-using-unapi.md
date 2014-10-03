---
title: "Setting up your OPAC for Zotero support using unAPI"
date: 2009-11-06
---

[unAPI](http://unapi.info) is a very simple protocol to let a machine know what other formats a document is available in. [Zotero](http://zotero.org) is a bibliographic management tool (like Endnote or Refworks) that operates as a Firefox plugin. And it speaks unAPI.

Let's get them to play nice with each other!

### How's it all work?

  1. Zotero looks for a well-constructed &lt;link&gt; tag in the head of the page
  2. It checks the document on the other side of that link to see what formats are offered, and picks one to use. No, you can't decide which one it uses. *It* picks.
  2. Zotero then looks for IDs in the body of the page
  3. If both are found and everything seems kosher, Zotero will offer the option to import some or all of the records. 

### What you'll need

  1. An OPAC whose output you can futz with
  2. Access to an individual record's ID in that output
  3. A URL based on the ID that gives an RIS representation of the records
  4. A screwdriver. Made with decent -- but not too expensive -- vodka and fresh orange juice.


### Yes. I'm cheating.

I have all those things already. Hence, this is easy for me. If you had to, say, write some sort of weird redirection script because IDs are not first-class citizens in your OPAC's URL scheme, or write an RIS export tool by hand, well, this will take you a bit longer.

### The process

#### 1. Build an upAPI target script

You need a script that'll do three things:

  1. With no arguments, return a list of available formats *in general*
  2. With one argument, id=&lt;ID&gt;, return a list of formats available *for that item*. This will likely be exactly the same as #1.
  3. With two arguments, id=&lt;ID&gt; &amp; format=&lt;FORMAT&gt;, return the record identified by &lt;ID&gt; in format &lt;FORMAT&gt;

Mine looks like this:


~~~php

  
  // id is of the form urn:bibnum:000000000
  
  $id = isset($_REQUEST['id'])? $_REQUEST['id'] : false;
  
  // Format, at this point, had better be 'ris'
  $format = isset($_REQUEST['format'])? $_REQUEST['format'] : false;

  // Got neither? Return the general list
  if (!($id || $format)) {
    header('Content-type: application/xml');
    echo '<?xml version="1.0" encoding="UTF-8"?>
    <formats>
      <format name="ris" 
              type="application/x-Research-Info-Systems" 
              docs="http://www.refman.com/support/risformat_intro.asp"/>
    </formats>
    ';
  exit;  
  }


  // Got just the id? Return formats for that ID
  if ($id && !$format) {
    header('Content-type: application/xml');
    echo '<?xml version="1.0" encoding="UTF-8"?>
    <formats id="' . $id . '">
      <format name="ris" 
              type="application/x-Research-Info-Systems" 
              docs="http://www.refman.com/support/risformat_intro.asp"/>
    </formats>
    ';  
  exit;  
  }


  // Otherwise...
 
  // Parse out the actual numeric part of the id from the urn:<typeOfNumber> prefix
  preg_match('/^urn:bibnum:(.*)$/', $id, $match);
  $actualID = $match[1];
  
  // Again: format had better be 'ris' because that's all I'm supporting at this point.
  header("Location: /Search/SearchExport?id=$actualID&method=$format", true, 302);

~~~

You can see that a &lt;format&gt; is a just a name, a mime-type, and an optional reference to documentation on the type.
  
I take advantage of my existing RIS export process in the redirect, at the bottom. I also built in the possibility that other types of numbers could come in -- I'm hard-coding 'bibnum' for the moment, but could allow, say, "oclc" or "isbn" or whatnot, too.

#### 2. Tell your OPAC where the script lives

You'll need a line in the &lt;head&gt; section of all your pages that might have an ID on them:
  
    <link rel="unapi-server" type="application/xml" title="unAPI" href="/unapi">

Everything should be left alone except for the actual *href*. 

#### 3. Add your IDs to the HTML

In the HTML of your page, you can add one or more tags of the form:

    <abbr class="unapi-id" title="urn:bibnum:000000002"></abbr>

(where the *title* of the &lt;abbr&gt; conforms to what you're expecting in your script). 

You can put stuff inside the &lt;abbr&gt; but you need not. On a single-record page, you should have (I would think) only one of these things. On a search results page, you may decide to not have any, or you may decide to have one for each search result.

#### 4. Final step

Drink your screwdriver. 

### Where can I see it? 

Well...here's the thing.

You can take a look at my test instance, [http://dueberb.vufind.lib.umich.edu/](http://dueberb.vufind.lib.umich.edu/) and play there. You can *not* see it in production, because there's a little problem.

Our old OPAC -- now dubbed [mirlyn-classic](http://mirlyn-classic.lib.umich.edu) -- had a custom translator written for it. And it worked fine, and that was great.

But now we've got this new software running at mirlyn.lib.umich.edu, and Zotero keeps on using the old translator no matter what you do. The only way to override it is to actually fire up sqlite3 and remove the conflicting entry from the zotero translators table. And then never update that table again. 

I've asked around about getting it fixed (changing the target URL for the old translator to point at mirlyn-classic) but it's Friday, and no one is around. Hopefully soon.
