# RubyGem Rakefile

## Usage
Add the Rakefile to your RubyGem project alter your gemspec like so

```ruby
require_relative 'lib/my-new-rubygem/version'

Gem::Specification.new do |s|
  s.name        = 'my-new-rubygem'
  s.version     = MyNewRubyGem::VERSION
  s.date        = '2016-10-12'
  s.summary     = 'Awesome!'
  s.description = ''
  s.authors     = ['Joe Bloggs']
  s.email       = 'joe.bloggs@default.com'
  s.homepage    = 'http://default.com'
  s.files       = [
    'lib/my-new-rubygem/version.rb',
  ]
  s.license     = 'MIT'
end
```
