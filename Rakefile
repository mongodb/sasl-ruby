# Copyright (C) 2009-2013 MongoDB Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

require "bundler"
Bundler.setup

$LOAD_PATH.unshift(File.expand_path("../lib", __FILE__))

require "rake"
require "rake/extensiontask"
require "rspec/core/rake_task"

def jruby?
  defined?(JRUBY_VERSION)
end

if jruby?
  require "rake/javaextensiontask"
  Rake::JavaExtensionTask.new do |ext|
    ext.name = "mongo_kerberos"
    ext.ext_dir = "src"
    ext.lib_dir = "lib"
  end
else
  require "rake/extensiontask"
  Rake::ExtensionTask.new do |ext|
    ext.name = "native"
    ext.ext_dir = "ext/mongo/kerberos"
    ext.lib_dir = "lib"
  end
end

require "mongo/kerberos/version"

def extension
  RUBY_PLATFORM =~ /darwin/ ? "bundle" : "so"
end

RSpec::Core::RakeTask.new(:spec)
RSpec::Core::RakeTask.new(:rspec)

if jruby?
  task :build => [ :clean_all, :compile ] do
    system "gem build mongo_kerberos.gemspec"
  end
else
  task :build => :clean_all do
    system "gem build mongo_kerberos.gemspec"
  end
end

task :clean_all => :clean do
  begin
    Dir.chdir(Pathname(__FILE__).dirname + "lib") do
      `rm native.#{extension}`
      `rm native.o`
      `rm mongo_kerberos.jar`
    end
  rescue Exception => e
    puts e.message
  end
end

task :ext_spec => :compile do
  ENV["WITH_EXT"] = "C"
  Rake::Task["rspec"].invoke
end

# Run bundle exec rake release with mri and jruby. Ex:
#
# rvm use 2.1.0@mongo_kerberos
# bundle exec rake release
# rvm use jruby@mongo_kerberos
# bundle exec rake release
task :release => :build do
  system "git tag -a v#{Mongo::Kerberos::VERSION} -m 'Tagging release: #{Mongo::Kerberos::VERSION}'"
  system "git push --tags"
  if jruby?
    system "gem push mongo_kerberos-#{Mongo::Kerberos::VERSION}-java.gem"
    system "rm mongo_kerberos-#{Mongo::Kerberos::VERSION}-java.gem"
  else
    system "gem push mongo_kerberos-#{Mongo::Kerberos::VERSION}.gem"
    system "rm mongo_kerberos-#{Mongo::Kerberos::VERSION}.gem"
  end
end

task :default => [ :clean_all, :spec, :ext_spec ]
