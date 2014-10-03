---
title: "Running Blacklight under JRuby"
date: 2009-11-18
---

I decided to see if I could get [Blacklight](http://projectblacklight.org/) working under [JRuby](http://jruby.org/), starting with running the test suite and working my way up from there.

There was much pain. Much, much pain. Exacerbated by my almost complete
lack of knowledge about what I was doing.

This is the procedure I eventually arrived at -- if there are places where I made trouble for myself, please let me know!

[And does anyone know how to get jruby's nokogiri to link to a different
libxml and stop with the crappy libxml2-version error message every time I 
run it under OSX???]

### Download jruby

Go to [jruby.org](http://jruby.org/) and download a binary distribution. Extract the tar.gz (or zip or whatever)

I'll put mine in ~/jruby. Or, at least that's what I'll tell you.

    tar xzf jruby-1.4.tar.gz

To avoid confusion, let's make _jrake_ an alias for _rake_ and add the jruby bin directory to the path

    cd ~/jruby/bin
    ln -s rake jrake
    export PATH=`pwd`:$PATH


### Download Blacklight

    git clone git://github.com/projectblacklight/blacklight.git

Again, well say that I put this in _~/blacklight/_

### Muck with Blacklight dependencies

Edit the file _init.rb_ to comment out references to _libxml_ and _ruby-xslt_, 
as well as _nokogiri_. My understanding is that the first two are used, at this point, only for the EAD stuff. Both rely on _libxml2_ which is a C-extension and hence unavailable to JRuby. 

Nokogiri gets pulled in during other installs and for some reason jrake will complain later on that it's got a wrong version or something. So, we'll just work without that particular net for now.

    #### File ~/blacklight/init.rb
    # config.gem 'libxml-ruby', :lib=>'libxml', :version=>'1.1.3'
    # config.gem 'ruby-xslt', :lib=>'xml/xslt', :version=>'0.9.6'
    # config.gem 'nokogiri', :version=>'1.3.3'

### Do some initial installs

    jgem install -v=2.3.4 rails 
    jgem install activerecord-jdbc-adapter jdbc-sqlite3 
                 activerecord-jdbcsqlite3-adapter ActiveRecord-JDBC 
    jgem install rcov -s http://gemcutter.org --no-rdoc --no-ri
    jrake
    jrake gems:install

### Edit the _config/database.yml_ file

...to change the adapter to _jdbcsqlite3_ for development and testing. 

### Edit the _databases.rake_ file

This one was harder to track down. The default rake task has hard-coded database names in the .rake file -- jdbcsqlite3 isn't included. I keep seeing things saying, "Oh, yeah, that's been fixed..." but, well, it wasn't for me. I had to do it by hand.

    edit ~/jruby/lib/ruby/gems/1.8/gems/rails-2.3.4/lib/tasks/databases.rake

You need to find everywhere there's a 

    when "sqlite", "sqlite3" # or when /^sqlite/ in one case

...and change it to

    when "sqlite", "sqlite3", "jdbcsqlite3"

Repeat for other databases you want to use (e.g., mysql). For the moment, since I'm only worried about running _jrake spec_, that's all I'm gonna do.

### Try again

    jrake
      Missing these required gems:
       mislav-hanna  = 0.1.11

OK. Not sure why that didn't come in before. Go head and add it.

    jgem install  mislav-hanna

### Migrate the databases

    jrake

The databases should migrate, and then it'll poop out because Solr didn't start.

### Fire up solr

Since we're running jruby, accessing the shell doesn't work. You'll have to fire up your test solr instance by hand.

    cd ~/blacklight/jetty
    java -Djetty.port=8888 -jar start.jar 2>log.jetty

### Try it again!

    cd ~/blacklight
    jrake spec
   
       ................................................................
       ................................................................
       ....F............................................................
       1)
       'ApplicationHelper Export EndNote should render the correct 
       EndNote text file' FAILED
       expected: "%0 Format\n%E Greer, Lowell. \n%E Lubin, Steven. \n%E Chase, Stephanie, \n%E Brahms, Johannes, \n%E Beethoven, Ludwig van, \n%E Krufft, Nikolaus von, \n%T Music for horn \n%I Harmonia Mundi USA, \n%C [United States] : \n%D p2001. \n",
      got: "%0 Format\n%C [United States] : \n%D p2001. \n%E Greer, Lowell. \n%E Lubin, Steven. \n%E Chase, Stephanie, \n%E Brahms, Johannes, \n%E Beethoven, Ludwig van, \n%E Krufft, Nikolaus von, \n%I Harmonia Mundi USA, \n%T Music for horn \n" (using ##)
    ./spec/helpers/application_helper_spec.rb:128:
    
    Finished in 15.519 seconds
    193 examples, 1 failure
  
I can live with that for the moment. Anyone know why that spec fails?
   
### Great! How about the features?

    jrake features
      (much output)
      
      59 scenarios (59 passed)
      434 steps (434 passed)
      0m51.186s

### And so...

...it appears that, at least on the surface, jruby is a viable platform for Blacklight so long as I don't actually need any of the _libxml_ stuff. In the next couple days I'll try and actually get it all up and running and see if I can break it.
