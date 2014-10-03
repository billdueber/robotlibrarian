---
title: "Sending MARC(ish) data to Refworks"
date: 2009-05-11
layout: post

---

Refworks has some <a href="http://www.refworks.com/DirectExport.htm">okish documentation</a> about how to deal with its callback import procedure, but I thought I'd put down how I'm doing it for our <a href="http://vufind.org/">vufind</a> install (<a href="http://mirlyn2-beta.lib.umich.edu/">mirlyn2-beta.lib.umich.edu</a>) in case other folks are interested.

The basic procedure is:

<ul>
	<li>Send your user to a specific refworks URL along with a callback URL that can enumerate the record(s) you want to import in a supported form</li>
	<li>Your user logs in (if need be) gets to her RefWorks page</li>
	<li>RefWorks calls up your system and requests the record(s)</li>
	<li>The import happens, and your user does whatever she want to do with them</li>
</ul>

Of course, there are lots of issues with doing this <em>well</em> (quick! Is this MARC record for a book? An edited book? Is it a journal, or a serial of some other sort? Who's the actual author/editor?), but doing it <em>at all</em> isn't so bad.

## The URL to send them to
This is the "Export this record" URL on my system:

~~~
http://www.refworks.com.proxy.lib.umich.edu/express/expressimport.asp?
vendor=[your system]&
filter=MARC+Format&
database=All+MARC+Formats&
encoding=65001
&url=[your callback URL]
~~~~

Note that the vendor variable should be a unique string (made up by your) for your <em>system</em>, not a larger entity (like the whole library or the institution).

The "MARC Format" filter we're using is <em>not</em> a filter for real MARC. It's a MARC-like delimited format (see <a target="marcish" href="http://mirlyn2-beta.lib.umich.edu/Record/000152772/Export?style=REF">an example from my catalog</a>).

Basically, you have three types of lines (but really, look at the example, 'cause it'll make everything a lot clearer):

LEADER
  : LEADER [one space] [leader text]

Control Field
  : [three-digit control tag] [four spaces] [data text]


Data Field
  : [three-digit data tag] [one space] [ind1] [ind2] [one space] [value of subfield a] [other subfield constructs]


...where [other subfield constructs] look like

~~~
  [pipe characeter][subfield code][subfield value]
~~~~


Notice that (a) there's no leading '\|a' before the subfield a value, and (b) there are no spaces between the pipe, the subfield code, and the subfield value for the non-code-a subfields.

Some easy PHP code to produce such a format is as follows. Note that I'm sending it as text (because it's <em>not MARC</em>) and UTF-8. If you're got MARC-8, you'll have to convert it before sending.


~~~PHP

      $m = $this->marcRecord;
      header('Content-type: text/plain; charset=UTF-8');

      echo 'LEADER ', $m->getLeader(), "\n";

      foreach ($m->getFields() as $tag => $val) {
        echo $tag;
        if ($val instanceof File_MARC_Control_FIELD) {
          echo '    ', $val->getData(), "\n";
        } else {
          echo ' ', $val->getIndicator(1),  $val->getIndicator(2), ' ';
          $subs = array();
          foreach ($val->getSubFields() as $code=>$subdata) {
            $line = '';
            if ($code != 'a') {
              $line = '|' . $code;
            }
            $subs[] = $line . $subdata->getData();
          }
          echo implode(' ', $subs), "\n";
        }
      }


~~~
