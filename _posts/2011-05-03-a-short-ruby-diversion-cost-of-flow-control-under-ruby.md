---
title: "A short ruby diversion: cost of flow control under Ruby"
date: 2011-05-03
tags: "ruby"
layout: post
slug: a-short-ruby-diversion-cost-of-flow-control-under-ruby

---

A couple days ago I decided to finally get back to working on [`threach`](https://github.com/billdueber/threach) to try to deal with problems it had -- essentially, it didn't deal well with non-local exits due to calls to `break` or even something simple like a `NoMethodError`.

[BTW, I think I managed it. As near as I can tell, `threach` version 0.4 won't deadlock anymore]

Along the way, while trying to figure out how threads affect the behavior of different non-local exits, I noticed that in some cases there was still work being done by one or more threads long after there was an exception raised.

I re-discovered something that a lot of people already know: `raise`/`rescue` under MRI is slow, and under JRuby can be *unbearably* slow. How slow?

Let's look at four simple blocks that exercise four different block exit strategies: `break`, `catch` and `throw`, `raise` with the normal single (or zero) arguments, as well as the three-argument version of `raise`.

<table class="data">
  <tr>
    <th>Simple break</th><th>Catch/Throw</th>
  </tr>
  <tr>
    <td>
      <pre lang="ruby">
range.each do |i|
  break
end
      </pre>
    </td>
    <td>
      <pre lang="ruby">
catch(:benchmarking) do  
 range.each do |i|
   throw(:benchmarking)
 end
end
      </pre>
    </td>
  </tr>
  <tr>
    <th>Raise (1 arg)</th><th>Raise (3 args)</th>
  </tr>
  <tr>
    <td>
      <pre lang="ruby">
 begin
   range.each do |i|
     raise StandardError
   end
 rescue
  # do nothing
 end
     </pre>
    </td>
    <td>
      <pre lang="ruby">
begin
  range.each do |i|
    raise StandardError, :hi, nil
  end
rescue
 # do nothing
end
      </pre>
    </td>
  </tr>
</table>


In each case, we immediately exit the block without doing any work; the idea is to measure how long it takes to break out for each case.

So....let's run them each 100K times and see what happens, shall we? Times are in seconds, averaged over two runs.


<table class="data" id="t">
  <tr>
    <th></th><th>Ruby 1.8</th><th>Ruby 1.9</th><th>JRuby</th><th>JRuby --1.9</th>
  </tr>  
  <tr><th>break</th>        <td>0.12</td><td>0.07</td><td>0.29</td> <td>0.21</td></tr>
  <tr><th>catch/throw</th>  <td>0.35</td><td>0.28</td><td>0.64</td> <td>0.48</td></tr>
  <tr><th>raise (1 arg)</th><td>1.78</td><td>2.10</td><td class="bad">26.60</td><td class="bad">22.06</td></tr>
  <tr><th>raise (3 arg)</th><td>1.85</td><td>2.13</td><td>0.45</td> <td>0.45</td></tr>
</table>

The first thing to note is that this is 100K iterations. Three of the strategies are fast enough that you'd have to work really, really hard to notice them.

In terms of speed, `raise (3 args)`, `catch/throw`, and `break` are fast enough that you shouldn't bother worrying about them (although you *should* choose the method that makes your code easy to understand).

The second things to note is *Holy Camoli!* JRuby is slow there!

[This Jira ticket](http://jira.codehaus.org/browse/JRUBY-5534) tells the tale: The creation of the backtrace is very, very expensive for JRuby. That `nil` at the end of the `raise (3 args)` call suppresses the creation of that backtrace, so the speed is fine.

Three things worth saying here:

* If you're using `raise/rescue` for flow control, *you're already doing it wrong.* Reserve exceptions for, well, exceptional conditions that are only going to be raised once or twice, not all the time.
* If you're writing code that, for some ungodly reason, is planning on raising a crapload of exceptions, use the three-arg version. I'm looking at you, gem authors.
* If you're writing your code without worrying about how it will work under multiple threads, well, please don't do that. Everyone has multi-core systems these days, and it's silly to not be able to use them. Plus, counting on Matz to never move to a VM with real threads is a big gamble.
