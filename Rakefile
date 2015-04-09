# -*- ruby -*-
# Rakefile: build ruby augeas bindings
#
# See AUTHORS for the list of authors
#
# Distributed under the GNU Lesser General Public License v2.1 or later.
# See COPYING for details

require 'rake/clean'
require 'rdoc/task'
require 'rake/testtask'
require 'rubygems/package_task'

GEM_NAME = 'augeas'
GEM_VERSION = '0.6.4'
EXT_CONF = 'ext/augeas/extconf.rb'
MAKEFILE = 'ext/augeas/Makefile'
AUGEAS_MODULE = 'ext/augeas/_augeas.so'
SPEC_FILE = 'augeas.spec'
AUGEAS_SRC = AUGEAS_MODULE.gsub(/\.so$/, '.c')

# Clean - remove all intermediate files / temporary files, do not touch final product.
CLEAN.include '**/*~',
              'ext/**/*.o',
              MAKEFILE,
              'ext/**/depend',
              'ext/**/mkmf.log'

# Clobber - return to original state.
CLOBBER.include AUGEAS_MODULE

file MAKEFILE => EXT_CONF do |t|
	Dir::chdir(File::dirname(EXT_CONF)) do
		unless sh 'ruby', File::basename(EXT_CONF)
			fail 'Failed to run extconf'
		end
	end
end

file AUGEAS_MODULE => [ MAKEFILE, AUGEAS_SRC ] do |t|
	Dir::chdir(File::dirname(EXT_CONF)) do
		unless sh "make"
			fail 'make failed'
		end
	end
end

desc 'Build the native library'
task :build => AUGEAS_MODULE

Rake::TestTask.new(:test) do |t|
	t.test_files = FileList['tests/tc_*.rb']
	t.libs = ['lib', 'ext/augeas']
end
task :test => :build


RDoc::Task.new do |rd|
	rd.main = "README.rdoc"
	rd.rdoc_dir = "doc/site/api"
	rd.rdoc_files.include("README.rdoc", "ext/**/*.[ch]","lib/**/*.rb")
end

PKG_FILES = FileList[
	"Rakefile", "COPYING","README.rdoc", "NEWS",
	"ext/**/*.[ch]", "lib/**/*.rb", "ext/**/MANIFEST", "ext/**/extconf.rb",
	"tests/**/*",
	"spec/**/*"
]

SPEC = Gem::Specification.new do |s|
	s.name = GEM_NAME
	s.version = GEM_VERSION
	s.email = "dot.doom@gmail.com"
	s.homepage = "https://github.com/dotdoom/augeas"
	s.summary = "Ruby bindings for augeas"
	s.authors = File.read('AUTHORS').lines.grep(/^  /).map { |a| a[/[^<]+/].strip }.uniq
	s.files = PKG_FILES
  s.licenses = ['LGPL-2.1+']
	s.required_ruby_version = '>= 1.8.7'
	s.extensions = "ext/augeas/extconf.rb"
	s.description = "A fork of ruby-augeas (bindings for augeas) with exceptions support."
	s.add_development_dependency "rake"
	s.add_development_dependency "rdoc"
end

Gem::PackageTask.new(SPEC) do |pkg|
	pkg.need_tar = true
	pkg.need_zip = true
end

task :default => [:build, :test]
