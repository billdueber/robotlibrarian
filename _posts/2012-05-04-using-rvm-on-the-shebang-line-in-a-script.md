---
title: "Ruby sidebar: Using rvm on the shebang (#!) line in a script"
date: 2012-05-04
---

Just throwing this up here because I didn't find it elsewhere.

I want to run ruby scripts from the command line or in a cronjob, and I do *not* want to always have to type "ruby scriptname".



*But*, I use [rvm](https://rvm.io/). I want to run a particular ruby, maybe identified by an alias, maybe with a specific gemset.

It turns out you can use the `env` program with `rvm do` to accomplish this.


~~~ruby

    #!/usr/bin/env  rvm 1.9 do ruby
    
    require 'mygem'
    o = MyGem.new
    # blah blah blah

~~~

In this example, `1.9` is the name of the ruby (actually, an rvm alias) I want to use, and it could just as easily specify a gemset as well (e.g., 1.9@mygems).

If you're running in cron, don't forget you need to load the environment variables first. Here I use the bash `.` command to source my `.bashrc`.


~~~

  54 9-16 * * 1-5 . /Users/dueberb/.bashrc; /Users/dueberb/bin/exercise

~~~~


Nothing fancy, but worth knowing.
