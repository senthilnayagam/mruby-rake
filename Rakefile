# Best to set MRUBY_PATH enviroment variable
# - or -
# Change the default path to match the path to your machine.
#
# assumes mrbgems root is [mruby_path]/mrbgems
def configure args
	$mrb_path = ENV['MRUBY_PATH'] || File.expand_path("../mruby")
	$mrbc = File.join($mrb_path,"/bin/mrbc")

	$gems_active = args[:use_gems] == "true" ? true : false

	$gem_path = File.join($mrb_path,"mrbgems")
	$gem_init = File.join($gem_path,"gem_init.a")

  if $gems_active
	a = []
	a1 = []

	get_active_gems($gem_path).each_pair do |k,v|
	  a << File.read(v[:ld_flags]).strip
	  a1 << v[:lib]
	end

	$gems_list = a1.join(" ")
	$gems_ld_flags = a.join(" ")
  end
end

# find active gems and get info about them
def get_active_gems gem_path
  h = {}
  File.open(gem_path+"/GEMS.active").readlines.each do |g|
    g=g.strip
    gem_lib = Dir.glob(g+"/*-gem.a")[0]
    gem_ldflags = File.join(g,"gem-ldflags.tmp")
    h[g]={:lib=>gem_lib,:ld_flags=>gem_ldflags}
  end
  h
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
  if $gems_active
    cmd2 = "gcc -o ./#{args[:program_name]} #{args[:program_name]}.o #{$mrb_path}/lib/libmruby.a -lm #{$gem_init} #{$gems_list} #{$gems_ld_flags}";  
  else
    cmd2 = "gcc -o ./#{args[:program_name]} #{args[:program_name]}.o #{$mrb_path}/lib/libmruby.a -lm"
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
