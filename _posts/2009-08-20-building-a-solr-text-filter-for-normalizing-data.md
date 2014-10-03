---
title: "Building a solr text filter for normalizing data"
date: 2009-08-20
layout: post

---

[Kind of part of a continuing series on our VUFind implementation; more of a sidebar, really.]

In my [last post](http://robotlibrarian.billdueber.com/easy-solr-types-for-library-data/) I made the case that you should put as much data normalization into Solr as possible. The built-in text filters will get you a long, long way, but sometimes you want to have specialized code, and then you need to build your own filter.

**Huge Disclaimer**: _I'm putting this up not because I'm the best person to do so, but because it doesn't look as if anyone else has. I don't know what I'm doing. I don't know why the code I'm showing below is the way it is, and if anyone would like to make it better, that'd be great. This is basically just a lot of pattern-matching on my part._

[A second disclaimer: I haven't actually built this into Solr yet, although I've done some simple testing on the ISBN-13 checksum code.  I'll remove this disclaimer when I get a chance to actually index some data with it.]

## The Setup: An ISBN-10 to ISBN-13 converter

Last time, I said I didn't know why I hadn't put together an ISBN longifier yet. So let's walk through it.

This is a lot easier than most things in that I'm assuming we're going to be getting exactly one token
to work with (via the _KeywordTokenizer_) and can just work on it with impunity.

If you'd like to follow along, get the [solr source via svn](http://svn.apache.org/repos/asf/lucene/solr/trunk) on a machine
with java and ant. And junit, I think.

##Where to put stuff

Of all the black magic associated with doing this, figuring out how to actually make it build is the part that's probably easiest for Java-heads and the most confusing to the rest of us. Anyone attempting this sort of thing should probably get a good grounding in how Solr is set up and how its build system works before doing anything else.

Me? I cheated.

I basically just copied the directory structure of another project in the `config` directory in the solr root (looks like maybe it was velocity), did some tiny modifications to the `build.xml` file to change the name of the project, renamed the '.pom' file and edited *it* in the obvious ways, and followed the copied directory structure to figure out where to put my files.

And then it worked. And I didn't ask any question, and metaphorically just backed away slowly with a nonchalant look on my face. Of course, if you know what you're doing with java and ant, I'm sure there are better ways.

For the record, the directory in solr/config/umichnormalizers (where I put this stuff) would look something like this by the end of this project:


~~~

./target/
./build.xml

./src/main/java/edu/umich/lib/normalizers/ISBNLongifier.java
./src/test/java/edu/umich/lib/normalizers/ISBNLongifier.java

./src/main/java/edu/umich/lib/solr/analysis/ISBNLongifierFilter.java
./src/main/java/edu/umich/lib/solr/analysis/ISBNLongifierFilterFactory.java


~~~~


You then just run `ant` in your config directory to generate a .jar file that can be put in solrmarc's `lib` directory or (I think) jetty's `lib` directory. You can also just run `ant dist` at the solr root level to get a .war file with your stuff embedded.

## The converter


First, you just need some basic code to actually do the conversion. I'm sure this is hideously inefficient, but probably not as inefficient as the actual filter I'll be producing in a minute.

We take in a string. If it looks like it might have a 10-digit ISBN in it (possibly with dashes or periods as delimiters), extract it, do the conversion to an ISBN-13, and return that as a 13-character string (e.g., no dashes or whatnot).

Note that I'm not working hard to determine if it's an ISBN -- this isn't designed to try to pull an ISBN from random text. The hope is that by the time you get this far you've already got a pretty good idea that you've got an ISBN on your hands. I'm also not checking to see if the incoming ISBN is valid in any way; that's left as an exercise for the dilligent reader.


~~~java

package edu.umich.lib.normalizers;
import java.util.regex.*;

public class ISBNLongifier {

  // dashes and dots are acceptable delimiters. Should we add spaces??
  private static String  ISBNDelimiiterPattern = "[\-\.]";

  // Look for a string of nine digits followed by another digit or an X
  private static Pattern ISBNPattern = Pattern.compile("^.*?(\d{9})[\dXx].*$");

  public static Boolean matches(String isbn)  throws IllegalArgumentException {
    isbn = isbn.replaceAll(ISBNDelimiiterPattern, "");
    Matcher m = ISBNPattern.matcher(isbn);
    return m.matches();
  }

  public static String longify(String isbn) {
    isbn = isbn.replaceAll(ISBNDelimiiterPattern, "");
    Matcher m = ISBNPattern.matcher(isbn);
    if (!m.matches()) {
      throw new IllegalArgumentException(isbn + ": Not an ISBN");
    }

    String longisbn = "978" + m.group(1);
    int[] digits = new int[12];
    for (int i=0;i<12;i++) {
      digits[i] =  new Integer(longisbn.substring(i, i+1));
    }

    Integer sum = 0;
    for (int i = 0; i < 12; i++) {
      sum = sum + digits[i] + (2 * digits[i] * (i % 2));
    }

    // Get the smallest multiple of ten > sum
    Integer top = sum + (10 - (sum % 10));
    Integer check = top - sum;
    if (check == 10) {
      return longisbn + "0";
    } else {
      return longisbn + check.toString();
    }
  }
}

~~~

The Factory Object
------------------

Next is a boilerplate factory object. The only change will be the package you put it in, and the last method's name and return value.


~~~java

package edu.umich.lib.solr.analysis;
import java.util.Map;
import org.apache.solr.analysis.BaseTokenFilterFactory;
import org.apache.lucene.analysis.TokenStream;

public class ISBNLongifierFilterFactory extends BaseTokenFilterFactory {
  Map<String,String> args;

  public Map<String,String> getArgs()
  {
    return args;
  }
  public void init(Map<String,String> args)
  {
    this.args = args;
  }
  public ISBNLongifierFilter create(TokenStream input)
  {
    return new ISBNLongifierFilter(input);
  }
}


~~~


## The actual filter

And, finally, the filter class. You'll notice that I'm catching any illegal argument error and just returning the input unchanged. So anything that comes through that *isn't* an ISBN just gets passed along.


~~~java

package edu.umich.lib.solr.analysis;

import edu.umich.lib.normalizers.ISBNLongifier;
import org.apache.lucene.analysis.Token;
import org.apache.lucene.analysis.TokenFilter;
import org.apache.lucene.analysis.TokenStream;
import java.util.regex.*;
import java.io.IOException;

public final class ISBNLongifierFilter extends org.apache.lucene.analysis.TokenFilter {

  public ISBNLongifierFilter(TokenStream in) {
    super(in);
  }

  public Token next() throws IOException {
    return normalize(this.input.next());
  }

  public Token next(Token result) throws IOException {
    return normalize(this.input.next());

  }

  public Token normalize(Token t) {
    if (null == t || null == t.termBuffer() || t.termLength() == 0) {
      return t;
    }
    String val = new String(t.termBuffer());
    try {
      t.setTermBuffer(ISBNLongifier.longify(val));
      return t;
    } catch (IllegalArgumentException e) {
       // pass it through unchanged
      return t;
    }
  }
}

~~~

## How to use it

Assuming you've managed to get it built into Solr and then deployed, just define it as a type in your `schema.xml`:


~~~xml

  <fieldType name="isbnlongifier" class="solr.TextField"  omitNorms="true">
    <analyzer>
      <tokenizer class="solr.KeywordTokenizerFactory"/>
      <filter class="edu.umich.lib.solr.analysis.ISBNLongifierFilterFactory"/>
    </analyzer>
  </fieldType>

  # and later...

  <field name="isbn" type="isbnlongifier" indexed="true" stored="false" multiValued="true"/>

~~~

## Conclusion

There it is. The rocket science is all hidden behind the import statements. My understanding is that casting the token value to/from Strings makes things horribly inefficient, but I'm pretty sure I've got bigger bottlenecks to tackle before worrying about this.
