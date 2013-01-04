mruby-rake
==========

a rake task to complie your ruby code into a executable binary


    rake -D compile

    rake compile[program_name,ruby_source,use_gems]
      Compiles all ruby files in pwd into an executable.
      option: program_name,   the name of the resulting executable
              ruby_source,    path to ruby files to compile
    	      use_gems,       pass 'true' if mruby compiled with mrbgems


    rake compile[hello,./,false]  # the last argument, false, is implicit and does not need passed
                                  # pass 'true' to link against mrbgems

    ./hello                       # execute


want to compile something else, first clean the files with "rake clean"


Features 
===============
    Compiled programs support passing argumentsfrom the command-line to ARGV
    (initial) Support of (MRB)GEMS
    Assembles source file from *.rb files in path specified ...
    (order is alpha-numeric, naming convention must match require order)


pre-requisites
===============


clone mruby in a repository and update the rakefile with the path

    git clone git://github.com/mruby/mruby.git
    cd mruby
    pwd
    make

read more about compiling mruby and using it here

https://github.com/mruby/mruby

https://speakerdeck.com/matt_aimonetti/mmmm-dot-mruby-everywhere-and-revisiting-ruby

http://matt.aimonetti.net/posts/2012/04/25/getting-started-with-mruby/

http://merbist.com/2012/04/27/introduction-to-mruby/



Known Issues
============

limitations are related to mruby capabilites as it is a subset of ruby features
you cannot require files in your ruby file, add all together in a single file
there is no rubygems support



planned features
================

hope I can get the version to include C code and ruby together


License
========

it is not GPL for sure, you can assume MIT or Ruby license and that should be fine


like it hate it tell me on twitter @senthilnayagam
