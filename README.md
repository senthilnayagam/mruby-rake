mruby-rake
==========

a simple rake task to complie your ruby code into a executable binary

hacker news post to discuss it

http://news.ycombinator.com/item?id=4709279


modified version of code found here

https://gist.github.com/3850240/863acd980a0b29b0d86bd8cececb1d91b9b5bd83

thanks Matt Aimonetti
https://github.com/mattetti




    rake -T

    rake clean        # clean
    rake compile      # example
    rake compilefile  # compile file usage:  rake compilefile file=filename.rb

    rake compilefile file=hello.rb 
    compiled
    executing
    You can now run by running ./example


    ./example


want to compile something else, first clean the files with "rake clean"

for running your ruby files,please copy them to this folder and compile them


 



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



what I dont know
================

how to pass parameters to the compiled ruby app, need to try it



License
========

it is not GPL for sure, you can assume MIT or Ruby license and that should be fine


like it hate it tell me on twitter @senthilnayagam

