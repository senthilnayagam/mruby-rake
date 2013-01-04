require 'rubygems'
require 'rake'
require 'pathname'

module MRuby
  module Gem
    class << self
      attr_accessor :processing_path
    end

    class Specification
      include Rake::DSL

      attr_reader :build
      attr_accessor :name, :dir

      def self.attr_array(*vars)
        attr_reader *vars
        vars.each do |v|
          class_eval "def #{v}=(val);@#{v}||=[];@#{v}+=[val].flatten;end"
        end
      end

      attr_array :licenses, :authors
      alias :license= :licenses=
      alias :author= :authors=
      attr_array :cflags
      attr_array :mruby_cflags, :mruby_includes, :mruby_ldflags, :mruby_libs

      attr_array :rbfiles, :objs
      attr_array :test_objs, :test_rbfiles
      attr_accessor :test_preload
   
      @@instances = {}
      def self.get_gem gem
        @@instances[gem]
      end
      def self.gems
        @@instances
      end
      def initialize(name, &block)
        @name = name
        @@instances[name] = self
        #@build = MRuby.build
        @dir = Gem.processing_path
        @cflags, @cxxflags, @objcflags, @asmflags = [], [], [], []
        @mruby_cflags, @mruby_ldflags, @mruby_libs = [], [], []
        @mruby_includes = ["#{dir}/include"]
        @rbfiles = Dir.glob("#{dir}/mrblib/*.rb")
        @objs = Dir.glob("#{dir}/src/*.{c,cpp,m,asm,S}").map { |f| f.relative_path_from(@dir).to_s.pathmap("#{build_dir}/%X.o") }
        @test_rbfiles = Dir.glob("#{dir}/test/*.rb")
        @test_objs = Dir.glob("#{dir}/test/*.{c,cpp,m,asm,S}").map { |f| f.relative_path_from(dir).to_s.pathmap("#{build_dir}/%X.o") }
        @test_preload = 'test/assert.rb'

        instance_eval(&block)
        end
        def build_dir;end
     end
  end
end

Dir.glob("#{ENV['MRUBY_PATH']}/build/mrbgems/*/mrbgem.rake").each do |f|
  load f
end

# Best to set MRUBY_PATH enviroment variable
# - or -
# Change the default path to match the path to your machine.
#
# assumes mrbgems root is [mruby_path]/mrbgems
def configure args
	$mrb_path = ENV['MRUBY_PATH'] || File.expand_path("../mruby")
	$mrbc = File.join($mrb_path,"/build/host/bin/mrbc")
end


def gem_ld_flags
  a = []
  MRuby::Gem::Specification.gems.each_pair do |n,g|
    a << g.mruby_ldflags.join(" ")
  end
  a.join(" ")
end


def wrapper_code(insert=nil)
  code = <<-EOF
#include "mruby.h"
#include "mruby/irep.h"
#include "mruby/proc.h"
#include "mruby/array.h"

  #{insert}

int
main(int argc, char **argv)
{
  /* new interpreter instance */
  mrb_state *mrb;
  mrb = mrb_open();

  int n = -1;
  int i;

  mrb_value ARGV;
  ARGV = mrb_ary_new_capa(mrb, argc);
  
  // start at one to get rid of file name
  for (i = 1; i < argc; i++) {
    mrb_ary_push(mrb, ARGV, mrb_str_new(mrb, argv[i], strlen(argv[i])));
  }
  
  mrb_define_global_const(mrb, "ARGV", ARGV);  

  /* read and execute compiled symbols */
  int irep = mrb_read_irep(mrb, ruby_bytecode);
  mrb_run(mrb, mrb_proc_new(mrb, mrb->irep[irep]), mrb_top_self(mrb));

  mrb_close(mrb);

  return 0;
}
EOF
end

desc """Compiles all ruby files in pwd into an executable.
option: program_name,   the name of the resulting executable
\t    ruby_source,    path to ruby files to compile
\t    use_gems,       pass 'true' if mruby compiled with mrbgems"""

task :compile, :program_name, :ruby_source, :use_gems do |t,args|
  args.with_defaults(:program_name=>'example', :use_gems => "false", :ruby_source => File.expand_path("./"))
  
  configure(args)
  
  # track generated files for clean
  File.open("generated.list",'w') do |f|
    f.puts "#{args[:program_name]}.c"
    f.puts "#{args[:program_name]}.o"
    f.puts "#{args[:program_name]}"
    f.puts "temp.rb"
    f.puts "temp.c"
 
  end
  
  # make one file from all the ruby source
  File.open("temp.rb", "w"){|f|
    Dir.glob(args[:ruby_source]+"*.rb").sort.each do |r|
      f.puts "# File: #{r}"
      f << File.read(r)
    end
  }

  # creates the .c file containing the bundled_ruby char array for the bytecode
  mrbc = "#{$mrbc} -Bruby_bytecode ./temp.rb"
  puts mrbc
  raise "MRBC CompileError"unless system(mrbc)

  example_file = wrapper_code(File.read("./temp.c"))
  File.open("#{args[:program_name]}.c", 'w'){|f| f << example_file}

  cmd = "gcc -I#{$mrb_path}/src -I#{$mrb_path}/include -c #{args[:program_name]}.c -o #{args[:program_name]}.o"
  if args[:use_gems]
    cmd2 = "gcc -o ./#{args[:program_name]} #{args[:program_name]}.o #{$mrb_path}/build/host/lib/libmruby.a -lm #{gem_ld_flags}";  
  else
    cmd2 = "gcc -o ./#{args[:program_name]} #{args[:program_name]}.o #{$mrb_path}/build/host/lib/libmruby.a -lm"
  end

  puts cmd
  raise "CompileError" unless system("#{cmd}")

  puts cmd2
  raise "LinkError" unless system("#{cmd2}")

  print "compiled\n"
  print "executing\n"
  print "You can now run by running ./#{args[:program_name]}\n"
end

desc "clean"
task :clean do
  puts "cleanup ..."
  File.open("generated.list").readlines.each do |f|
    f=f.strip
    `rm #{f}`
  end
  `rm generated.list`
end

task :default => ["compile"]
