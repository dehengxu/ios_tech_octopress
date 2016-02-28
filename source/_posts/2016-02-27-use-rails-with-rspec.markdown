---
layout: post
title: "Use rails with Rspec"
date: 2016-02-27 23:47:04 +0800
comments: true
published: true
sharing: true
footer: true
categories:
---

First:

Add to `Gemfile`

```
group :test do
    gem 'selenium-webdriver'#, '2.0.0'
    gem 'capybara', '2.1.0'
    gem 'test-unit'
end
```

Second modify file named `spec_helper.rb`

```
require 'capybara/rspec'
require 'capybara/rails'

RSpec.configure do |config|
    ...

    config.include Capybara::DSL

    ...
end
```


Then create Unit-testing source with this command `rails generate integration_test static_pages`

```
require 'spec_helper'

describe "Static Pages" do
  describe "Home pages" do
    it "shoul be " do
      # Run the generator again with the --webrat flag if you want to use webrat methods/matchers
      # These are test-unit framework syntaxs
    #   get static_pages_index_path
    #   response.status.should be(200)
      # These are feature syntaxs supplied by Capybara.
        visit '/static_pages/help'
        expect(page).to have_content 'Sample App'
    end
  end
end

```

At last, you could run test

```
bundle exec rspec spec
```
