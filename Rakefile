require 'rake/testtask'
require 'rake/clean'
require 'rdoc/task'
require 'rubygems/package_task'

spec = Gem::Specification.load(Dir.glob("*.gemspec").first)

task :default => :test

Rake::TestTask.new do |t|
  t.pattern = "test/**/test*.rb"
end

gem_pkg = File.join('pkg', "#{spec.name}-#{spec.version}.gem")
deb_pkg = File.join('pkg', "ruby-#{spec.name}_#{spec.version}_all.deb")

namespace :test do
  task :verbose do
    if ENV['TESTOPTS'].nil?
      ENV['TESTOPTS'] = "--verbose"
    else
      ENV['TESTOPTS'] += " --verbose"
    end

    Rake::Task[:test].invoke
  end

  task :gem => 'build:gem' do
    require 'tmpdir'

    Dir.mktmpdir('gem-test-') do |dir|
      ENV['GEM_HOME'] = dir
      ENV['GEM_PATH'] = [dir ,`gem env path`].join(':')

      sh("gem install --no-doc #{gem_pkg}")

      Rake::TestTask.new(:gem_test) do |t|
        t.libs = ""
        t.pattern = "test/**/test*.rb"
      end
      Rake::Task[:gem_test].invoke
    end
  end
end

task :build => 'build:gem'
namespace :build do
  task :gem => gem_pkg
  task :deb => deb_pkg
end

Gem::PackageTask.new(spec) do |pkg|
  spec.metadata = {
    'git-commit' => `git rev-parse --verify HEAD`.chomp
  }
end

file deb_pkg => gem_pkg do
  sh("fpm -s gem -t deb --maintainer \"#{spec.authors.join(', ')} <#{spec.email}>\" --gem-package-name-prefix ruby -d systemd-container -d debootstrap -p pkg #{gem_pkg}")
end
task :package => deb_pkg

task :install => gem_pkg do
  sh("gem install --user-install #{gem_pkg}")
end

task :uninstall do
  sh("gem uninstall --user-install #{spec.name} -v #{spec.version}")
end

namespace :release do
  def _release(spec, type)
    unless (`git rev-parse --abbrev-ref HEAD`.chomp == 'master')
      puts 'ERROR: Not on master branch.'
      exit(1)
    end

    major, minor, bugfix, pre = spec.version.to_s.split('.')

    major  = major.to_i
    minor  = minor.to_i
    bugfix = bugfix.to_i

    case(type)
    when :major
      major  += 1
      minor   = 0
      bugfix  = 0
    when :minor
      minor  += 1
      bugfix  = 0
    when :bugfix
      bugfix += 1
    else
      raise ArgumentError.new("Invalid type #{type}")
    end

    version = [major,minor,bugfix].join('.')

    version_file = spec.files.select do |f|
      f.include?(File.join(spec.name, 'version.rb'))
    end.first

    content = String.new()
    File.open(version_file) do |f|
      f.each_line do |l|
        if l.match(/^\s+VERSION\s*=/)
          content << l.sub(spec.version.to_s, version)
        else
          content << l
        end
      end
    end

    File.write(version_file, content)

    sh("git add #{version_file}")
    sh("git commit -m 'Release version #{version}'")
    sh("git tag -a #{version} -m 'Release version #{version}'")
  end

  task :bugfix => :test do
    _release(spec, :bugfix)
  end

  task :minor => :test do
    _release(spec, :minor)
  end

  task :major => :test do
    _release(spec, :major)
  end
end

RDoc::Task.new do |rdoc|
  rdoc.rdoc_files.include("lib/**/*.rb")
end

task :tags do
  sh('ctags -R --exclude=.git --languages=ruby')
end
