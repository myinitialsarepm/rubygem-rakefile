require 'rake/testtask'
require 'rake/clean'

load Dir.glob("*.gemspec").first

task :default => :test

Rake::TestTask.new do |t|
  t.pattern = "test/test*.rb"
end

task 'test:verbose' do
  ENV['TESTOPTS'] = "--verbose"
  Rake::Task[:test].invoke
end

task :build => 'build:gem'
namespace :build do
  task :gem => "#{SPEC.name}-#{SPEC.version}.gem"
  task :deb => "rubygem-#{SPEC.name}_#{SPEC.version}_all.deb"
end

file "#{SPEC.name}-#{SPEC.version}.gem" do
  sh("gem build #{SPEC.name}.gemspec")
end
CLOBBER << "#{SPEC.name}-#{SPEC.version}.gem"

file "rubygem-#{SPEC.name}_#{SPEC.version}_all.deb" => "#{SPEC.name}-#{SPEC.version}.gem" do
  sh("fpm -s gem -t deb #{SPEC.name}-#{SPEC.version}.gem")
end
CLOBBER << "rubygem-#{SPEC.name}_#{SPEC.version}_all.deb"

task :install => "#{SPEC.name}-#{SPEC.version}.gem" do
  sh("gem install --user-install #{SPEC.name}-#{SPEC.version}.gem")
end

task :uninstall do
  sh("gem uninstall --user-install #{SPEC.name} -v #{SPEC.version}")
end
