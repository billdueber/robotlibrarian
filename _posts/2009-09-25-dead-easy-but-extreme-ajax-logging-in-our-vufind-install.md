---
title: "Dead-easy (but extreme) AJAX logging in our VuFind install"
date: 2009-09-25
layout: post

---

One of the advantages of having complete control over the OPAC is that I change things pretty easily. The downside of that is that we need to know *what* to change.

Many of you that work in libraries may have noticed that data are not necessarily the primary tool in decision-making. Or, say, even a part of the process. Or even thought about hard. Or even considered.

For many decisions I see going on in the library world, the primary motivator is the *anecdote*. In fact, to be honest, the primary driver is the *faculty anecdote*. Those cliched three curmudgeonly old faculty members invariably have huge influence over systems and interfaces that will be used by 40K undergraduates. The tiny percentage of weirdos that actually talk to reference librarians end up wielding enormous power compared to the untold masses that don't.

### Enter the dragon...er...log.

So...I'm logging everything. EVERYTHING. Everything I can think of, anyway, and that doesn't slow things down too far.

I've got a simple database table set up with the following columns:

  * incrementing integer (solely for innodb's efficiency needs)
  * sessionid
  * action
  * data1, data2, and data3 (all of these are action-dependent)
  * logweekday, logdate, logtime (instead of a single timestamp for easy and efficient queries)

And that's it. I've had it running (initially with only a few actions) for two weeks and have on the order of 300,000 rows in it at this point. Obviously, at some point I'll have a better idea of which data I actually care about and things will get slimmed down a little bit. But for now, it's fun having that all around.

Common log events include an action, and usually at least one other piece of data. Stuff like:

  * start-a-new-session with IP Address
  * simple-search with search-index, searchstring
  * choose-a-facet with facet-index, facet-value, position-on-list
  * view-a-full-record with recordID, search-result-number
  * click on an electronic resource link to proquest/google/hathitrust/whatever

...etc. I track adding and removing things from the selected items set and the user's favorites, exporting to email or refworks or whatnot, logging in and out, clicking on the author's name or a LCSH subject in the full record view, picking a "similar item" from the eponymous list, clicking on the spelling suggestion and the prev/next buttons, etc. Currently I'm logging 78 events.

[Note: by "search result number" I mean the enumeration of that record in that specific search set. So, the top result is #1. The first result on page two is #21]

### What do I think I'm gonna learn?

I'm not exactly sure. Of course, I can get all the basics -- how much traffic and what people are searching for -- but there's the possibility of other stuff. Things like:

  * Do people actually use the [prev/next, facets, facets-below-the-fold, items past the third page, etc]
  * Seriously, is anyone using the boolean searches and wildcards on the basic search page? Are any of them using IP addresses from outside the staff subnet? If not, can I please please please start using DisMax???
  * What facets are most popular? Do people hit the little "more" button to expand the list of facet values from 6 to 30?
  * What's the average search result number of a record chosen for a full-record view for each search index (perhaps an indicator of how well the relevancy-ranking is working?)
  * Looking at all the full-record displays, what are the patterns for those records (e.g., break down by callnumber prefix, or by our "Academic Discipline" subject)

I've got a lot to learn about stats, and user tracking, and clickpath analysis, but dammit I'll have *data* and I'm not afraid to use it!

[Er...them. Not "it". Data are a "them."  Always feels weird to me to refer to data in the plural, but I'm forcing myself to do so these days.]

### What's the server implementation?

I already mentioned the database. I've got a little module called `ActivityLog` that does three and a half things:

  1. Get the session id from the session
  2. Get logging information from the GET/POST or passed in as parameters
  3. Modify the parameters if need be (e.g., pull domain name out of an external URL). This is the half a thing.
  4. Stuff it into the database with appropriate timestamps.

And that's it.

### What's the client need to do?

I start off with the following rules:

  1. I want to be able to log damn near everything
  2. I can't degrade the user experience in a meaningful way just for logging
  3. I want to log outgoing links, too.
  4. I must must must have pretty, bookmarkable URLs.

Truth be told, some of the "client" stuff can be (and is) done on the server. When someone is, say, sending a record or set of records to RefWorks, the server knows everything it needs to know and I can just take care of logging as part of the regular request fulfillment.

But some stuff -- like the search result number, say -- are best taken care of from the browser. Easy enough, for the most part, esp. with form submissions and such.

The potentially-non-obvious part comes in with rule #3 -- I want pretty URLs. That means that the full display of record 123456789 is always going to be at `/Record/123456789` no matter what the user clicked on. Ditto with adding/removing facets and such -- the URL contains the resulting search, not the resulting search *plus which facet was removed or added*.

But -- see #1 -- I want to log damn near everything.

My solution -- and I know lots of people are doing this; this isn't rocket science -- is to fire off an AJAX post for the click events that I'm interested in, sending log data off to my server and then *not waiting for a return*. Just send the data and follow the link as if nothing had intervened. It degrades gracefully (although the rest of my VuFind doesn't, so that doesn't matter much) and it dead-easy to implement.  

### The actual javascript implementation

I long ago switched our VuFind stuff over to use jQueryuery, just because I like it and know it.

First thing is to use the templates to modify the links to have a particular class (`logit`) and a well-structured *ref* (pipe-delimited values).

So, a link from the title of a work on the search-results page to the individual record will look like this:

    <a ref="srrecview|{$record.id}||{$recordCounter}" href="/Record/{$record.id}" class="title logit">{$title}</a>

The *ref* attribute tells us that we're going to log the type of event (record view from the search results), the ID of the record, a null in the data2 column, and the search result number.

Then there's javascript to make all the magic happen:


~~~javascript



  function logit(a, args) {
    a = jQuery(a);

    // Allow the caller to pass in args, or get them from the ref attribute
    if (!args) {
      args = a.attr('ref').split('|');
    }

    jQuery.post(
      url_to_the_logging_method,
      {
        'lc' : args[0],
        'lv1': args[1] || '',
        'lv2': args[2] || '',
        'lv3': args[3] || ''
      }
    );
  }

  jQuery(document).ready(function() {
    jQuery('a.logit').live('click', function(e) {
      logit(this);
    });
  });  

~~~

The `logit` function just does a brain-dead post of the data in the *ref* attribute. We then bind that function to all anchors with the appropriate class, and we're done.

[Note the use of the jQuery `live` event -- this makes sure the event will be bound to stuff that comes in via AJAX after page load. Our links to Google Books, for example, come in like this.]

Since I'm not returning `false` from the logit function, the default action (actually follow the link) will fire -- without even waiting for the AJAX call to come back. Delay to the user is, hopefully, unnoticeable.

### Final words

This isn't all that smart. I should be doing more data-integrity stuff than I am, and of course someone could spoof my numbers if they wanted. But someone could spoof my stats just by hitting my normal catalog pages programatically, too, so there's no more risk involved, and I *do* log IPs.

And, of course, I get my pretty URLs, and most users (i.e., those not running firebug) will never notice anything.

I don't know that this would work for everyone, but so far it's working pretty well for us. I'll let you know if that continues in a post in a few weeks.
