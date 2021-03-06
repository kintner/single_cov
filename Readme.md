Actionable code coverage.

 - Easily add coverage tracking/enforcement for legacy apps
 - Get actionable feedback on every successful test run
 - Only 2-5% runtime overhead on small files compared to 50% for `SimpleCov`
 - No more PRs with bad test coverage
 - Branch coverage on ruby 2.5+ (disable via `branches: false`)

```Ruby
# Gemfile
gem 'single_cov', group: :test

# spec/spec_helper.rb ... load before loading rails / minitest / libraries
require 'single_cov'
SingleCov.setup :rspec

# spec/foobar_spec.rb ... add covered! call to every test file
require 'spec_helper'
SingleCov.covered!

describe "xyz" do ...
```

```Bash
rspec spec/foobar_spec.rb
......
114 example, 0 failures

lib/foobar.rb new uncovered lines introduced (2 current vs 0 configured)",
Uncovered lines:
lib/foobar.rb:22
lib/foobar.rb:23
```

### Minitest

Call setup before loading minitest.

```Ruby
SingleCov.setup :minitest
require 'minitest/autorun'
```

### Unfound target file

```Ruby
# change the guessed path
SingleCov.rewrite { |f| f.sub('lib/unit/', 'app/models/') }

# mark directory as being in app and not lib
SingleCov::APP_FOLDERS << 'presenters'
```

### Known uncovered

Prevent addition of new uncovered code, without having to cover all existing code.

```Ruby
SingleCov.covered! uncovered: 4
```

### Unconventional files

```Ruby
SingleCov.covered! file: 'scripts/weird_thing.rb'
```

### Forking vs RSpec

```Ruby
it "does not alert in forks" do
  fork { SingleCov.disable }
  expect(true).to eq true
end
```

### Checking usage

```Ruby
# spec/coverage_spec.rb
SingleCov.not_covered! # not testing any code in lib/

describe "Coverage" do
  it "does not let users add new untested code" do
    # option :tests to pass custom Dir.glob results
    SingleCov.assert_used
  end

  it "does not let users add new untested files" do
    # option :tests and :files to pass custom Dir.glob results
    # :untested to get it passing with known untested files
    SingleCov.assert_tested
  end
end
```

### Automatic bootstrap

Run this from `irb` to get SingleCov added to all test files.

```Ruby
tests = Dir['spec/**/*_spec.rb']
command = "bundle exec rspec %{file}"

tests.each do |f|
  content = File.read(f)
  next if content.include?('SingleCov.')

  # add initial SingleCov call
  content = content.split(/\n/, -1)
  insert = content.index { |l| l !~ /require/ && l !~ /^#/ }
  content[insert...insert] = ["", "SingleCov.covered!"]
  File.write(f, content.join("\n"))

  # run the test to check coverage
  result = `#{command.sub('%{file}', f)} 2>&1`
  if $?.success?
    puts "#{f} is good!"
    next
  end

  if uncovered = result[/\((\d+) current/, 1]
    # configure uncovered
    puts "Uncovered for #{f} is #{uncovered}"
    content[insert+1] = "SingleCov.covered! uncovered: #{uncovered}"
    File.write(f, content.join("\n"))
  else
    # mark bad tests for manual cleanup
    content[insert+1] = "# SingleCov.covered! # TODO: manually fix this"
    File.write(f, content.join("\n"))
    puts "Manually fix: #{f} ... output is:\n#{result}"
  end
end
```

Author
======
[Michael Grosser](http://grosser.it)<br/>
michael@grosser.it<br/>
License: MIT<br/>
[![Build Status](https://travis-ci.org/grosser/single_cov.png)](https://travis-ci.org/grosser/single_cov)
