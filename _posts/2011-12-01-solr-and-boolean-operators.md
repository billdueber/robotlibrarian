---
title: "Solr and boolean operators"
date: 2011-12-01
---

[Summary: **ALWAYS ALWAYS ALWAYS USE PARENTHESES TO GROUP BOOLEANS IN SOLR!!!**]

What does Solr do, given the following query?


~~~
  a OR b AND c
~~~~


I'll give you three guesses, but you'll get the first two wrong
and won't have any idea how to generate a third, so don't spend too much time on it. 

### Boolean algebra and operator precedence

Anyone who's had even a passing introduction to boolean alegebra knows that it specifies a strict order to how the operators are bound: NOT before AND before OR. So, one might expect the following grouping:


~~~

  a OR (b AND c)

~~~~


That's guess one. It's not how Solr does it.

### Left to right?

Some naive students, and at least one programming language (Smalltalk), do a simple left-to-right evaluation. So you might go with:


~~~

  (a OR b) AND c

~~~~


Nope. Wrong again.

### So what's left???

Excellent question. I don't know the code well enough to know what's going on underneath, but here's what we get under the lucene query parser.


~~~

    (b AND c)

~~~~


That's right. The first term is thrown away.(More correctly, the first term is deemed "optional").

### Do you let your users put AND/OR/NOT in their queries?

Hopefully, they don't know any boolean algebra. If they do, hopefully they use parentheses, or you parse it out for them. And if not, well, they're gonna be pretty damn confused.

### It gets weirder

I populated a fresh solr (3.5) index with all possible subsets of the strings "curly", "larry", "moe", and "shemp" (not Joe. Don't talk to me about Joe). There are 15 of them, from the one-item 'curly' to all four at once. 

I wrote a script to run a set of queries against the index under both lucene and edismax to see what I would get. In all cases the default lucene operator is 'AND' and the edismax mm parameter is set to 100% (equivalent to "all required").


~~~

        Lucene                    EDismax                  
  -------------------------------------------------------
  
  1. curly AND larry
        curly larry               curly larry              
        curly larry moe           curly larry moe          
        curly larry shemp         curly larry shemp        
        curly larry moe shemp     curly larry moe shemp    

  2. curly AND larry OR moe
        curly                     curly larry              
        curly larry               curly larry moe          
        curly moe                 curly larry shemp        
        curly shemp               curly larry moe shemp    
        curly larry moe                                    
        curly larry shemp                                  
        curly moe shemp                                    
        curly larry moe shemp                              

  3. curly OR larry AND moe
        larry moe                 larry moe                
        curly larry moe           curly larry moe          
        larry moe shemp           larry moe shemp          
        curly larry moe shemp     curly larry moe shemp    

  4. curly AND larry OR moe AND shemp
        curly moe shemp           curly larry moe shemp    
        curly larry moe shemp                              

  5. moe AND shemp OR curly AND larry
        curly larry moe           curly larry moe shemp    
        curly larry moe shemp
        

~~~~


Query 1 is as expected. Query 2 apparently reduces to just 'curly' under the lucene parser and 'curly AND larry' under edismax (and query 3 similarly reduces to the two AND'd words). Queries 4 and 5 are...well, you can look at the debugQuery output to see what it gets, but not **why**. And then tell me how to explain it to a user. 

### Where does this leave us?

The good news is that both lucene and edismax behave predictably when you use parentheses for grouping. So do that.

I'm generally not one to complain about open-source software, at least partially because I don't have the chops to do anything about it most of the time, but I don't understand how this could seem OK to anyone. There are a couple lucene Jira tickets ([Lucene-167](https://issues.apache.org/jira/browse/LUCENE-167) and [Lucene-1823](https://issues.apache.org/jira/browse/LUCENE-1823)) and a [2005 mailing list thread](http://www.mail-archive.com/java-user@lucene.apache.org/msg00008.html) denouncing the current behavior, but it persists.

Until the Solr/Lucene powers that be decide to tackle this, the rest of us will either have to write pre-parsers to make sure users get something sensible, or cripple our applications to disallow unrestricted boolean queries.
