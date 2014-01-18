# -*- ruby -*-
# Rakefile: build ruby augeas bindings
#
# Copyright (C) 2008 Red Hat, Inc.
#
# Distributed under the GNU Lesser General Public License v2.1 or later.
# See COPYING for details
#
# Bryan Kearney <bkearney@redhat.com>

require 'rake/clean'
require 'rdoc/task'
require 'rake/testtask'
require 'rubygems/package_task'

PKG_NAME='ruby-augeas'
GEM_NAME=PKG_NAME # we'd like 'augeas' but that makes RPM fail
PKG_VERSION='0.5.0'
EXT_CONF='ext/augeas/extconf.rb'
MAKEFILE="ext/augeas/Makefile"
AUGEAS_MODULE="ext/augeas/_augeas.so"
SPEC_FILE="ruby-augeas.spec"
AUGEAS_SRC=AUGEAS_MODULE.gsub(/.so$/, ".c")

#
# Building the actual bits
#
CLEAN.include [ "**/*~",
                "ext/**/*.o", AUGEAS_MODULE,
                "ext/**/depend" ]

CLOBBER.include [ "config.save",
                  "ext/**/mkmf.log",
                  MAKEFILE ]

file MAKEFILE => EXT_CONF do |t|
    Dir::chdir(File::dirname(EXT_CONF)) do
         unless sh "ruby #{File::basename(EXT_CONF)}"
             $stderr.puts "Failed to run extconf"
             break
         end
    end
end
file AUGEAS_MODULE => [ MAKEFILE, AUGEAS_SRC ] do |t|
    Dir::chdir(File::dirname(EXT_CONF)) do
         unless sh "make"
             $stderr.puts "make failed"
             break
         end
     end
end
desc "Build the native library"
task :build => AUGEAS_MODULE

#
# Testing
#
Rake::TestTask.new(:test) do |t|
    t.test_files = FileList['tests/tc_*.rb']
    t.libs = [ 'lib', 'ext/augeas' ]
end
task :test => :build


#
# Generate the documentation
#
RDoc::Task.new do |rd|
    rd.main = "README.rdoc"
    rd.rdoc_dir = "doc/site/api"
    rd.rdoc_files.include("README.rdoc", "ext/**/*.[ch]","lib/**/*.rb")
end

#
# Packaging
#
PKG_FILES = FileList[
  "Rakefile", "COPYING","README.rdoc", "NEWS",
  "ext/**/*.[ch]", "lib/**/*.rb", "ext/**/MANIFEST", "ext/**/extconf.rb",
  "tests/**/*",
  "spec/**/*"
]

DIST_FILES = FileList[
  "pkg/*.tgz", "pkg/*.gem"
]

SPEC = Gem::Specification.new do |s|
    s.name = GEM_NAME
    s.version = PKG_VERSION
    s.email = "augeas-devel@redhat.com"
    s.homepage = "http://augeas.net/"
    s.summary = "Ruby bindings for augeas"
    s.authors = [ "Bryan Kearney", "David Lutterkort" ]
    s.files = PKG_FILES
    s.autorequire = "augeas"
    s.required_ruby_version = '>= 1.8.1'
    s.extensions = "ext/augeas/extconf.rb"
    s.description = "Provides bindings for augeas."
    s.add_development_dependency "rake"
    s.add_development_dependency "rdoc"
end

Gem::PackageTask.new(SPEC) do |pkg|
    pkg.need_tar = true
    pkg.need_zip = true
end

task :default => [:build, :test]
