# Rails Tips

Surprising or less know things from the Ruby on Rails universe

#### How to DRY up my routes?

Perhaps [routing concerns](http://guides.rubyonrails.org/routing.html#routing-concerns)
can help you. Concern is like a mixin of routes that you can use inside other
resources and routes:

```ruby
concern :taskable do
  resources :tasks, :only => [:index, :create, :destroy]
end

resources :users, :concerns => :taskable
resources :managers, :concerns => :taskable
resources :admins, :concerns => :taskable
```

This is the same as:

```ruby
resources :users do
  resources :tasks, :only => [:index, :create, :destroy]
end

resources :managers do
  resources :tasks, :only => [:index, :create, :destroy]
end

resources :admins do
  resources :tasks, :only => [:index, :create, :destroy]
end

```

#### How to send a piece of HTML to a partial?

Do you need to send a piece of HTML to a partial? Perhaps the partial is used
on a number of places, but you need to show a different message from each place?
You can use Rails' `capture` helper:

```erb
<% personalized_message = capture do %>
  Hi <%= user.name %>! Welcome to the greatest collection of programming books
  in the world!
<% end %>

<%= render "greeting", :message => personalized_message %>
```

Read more about `capture` on the official Rails API
[documentation](http://api.rubyonrails.org/classes/ActionView/Helpers/CaptureHelper.html#method-i-capture).

#### How to Maximize Firefox in a Selenium Test?

Sometimes an end-to-end browser test will fail on a small screen, but pass when
it's executed on a large screen. One way to fix this is to maximize the Firefox
window that's executing the test. You can do that with this one-liner:

```ruby
page.driver.browser.manage.window.maximize
```

If you want to maximize the Firefox window in all scenarios that execute in Firefox,
you should check if the current test is executing in a real browser first. If you're
using Cucumber, you can add this to your `support/env.rb` file:

```ruby
Before do
  Capybara.current_driver == Capybara.javascript_driver
    page.driver.browser.manage.window.maximize
  end
end
```

#### How to accept a JavaScript dialog in Selenium tests?

Capybara has a neat method called
[accept_dialog](http://www.rubydoc.info/github/jnicklas/capybara/master/Capybara%2FSession%3Aaccept_alert)
that will do just what you need. Developers used to lean on different JavaScript
hacks to accept a JavaScript dialog, but the built in `accept_dialog` method is
the recommended way.

```ruby
accept_dialog("Are you sure?")
```

#### How to skip assets, helpers and specs when generating a controller?

I rarely need assets, helpers and specs for a new controller. And I don't like
leaving empty files all over the place. To skip generating those, you can use
the following options when you generate a new controller:

```bash
rails g controller Posts --no-assets --no-helper --no-controller-specs
```

You can also use `skip` instead of `no`:

```bash
rails g controller Posts --skip-assets --skip-helper --skip-controller-specs
```

If other developers from your team tend to skip those and you want to save some
typing, add the following to your `config/application.rb`:

```ruby
config.generators do |g|
  g.test_framework nil # skip test framework
  g.assets false
  g.helper false
end
```

For more options see [Generators](http://guides.rubyonrails.org/generators.html)
page on [Rails Guides](http://guides.rubyonrails.org/index.html).

#### How to wait for a JavaScript library to load in Selenium tests?

Sometimes you need to wait for a JavaScript library to be fully loaded before
executing any steps in Selenium tests. To do this, create a helper and make it
available in your test library of choice (RSpec, Cucumber...):

```ruby
module WaitForJqueryUjs
  def wait_for_jquery_ujs
    Timeout.timeout(Capybara.default_wait_time) do
      loop until jquery_ujs_loaded?
    end
  end

  def jquery_ujs_loaded?
    page.evaluate_script("jQuery.rails !== undefined")
  end
end
```

Then, in a test, you can use `wait_for_jquery_ujs` before any steps that require
the library:

```ruby
wait_for_jquery_ujs
fill_in "Name", :with => "John Doe"
click_button "Save"
```

You can use this pattern to inspect other things in JavaScript. However, you
should consider leaning on a visible element on a page, instead of executing
JavaScript code. For example, you application can show a loading indicator
until everything is fully loaded. In tests, you can wait until the loading
indicator goes away before executing any steps.

#### How to add ":type => :model" to all RSpec model specs?

If you're upgrading a gem that's using `:type => :model` metadata internally,
like shoulda-matchers, this can be useful. Just execute this command in your
shell:

```bash
find spec/models -iname "*.rb" -print | xargs sed -ri "s/describe ([A-Z].*) do/describe \1, :type => :model do/g"
```

#### How to configure a Rails engine to use RSpec instead of MiniTest when generating new files?

Add this to `engine.rb` in the engine:

```ruby
config.generators do |g|
  g.test_framework :rspec
end
```

#### How to configure a Rails engine not to generate assets or helpers?

Add this to `engine.rb` in the engine:

```ruby
config.generators do |g|
  g.assets false
  g.helper false
end
```

#### Tips for debugging flaky Cucumber scenarios

To print the last page HTML when a scenario fails, add this after hook to your
`env.rb` file:

```ruby
After do |scenario|
  return unless scenario.failed?

  puts "********* PAGE: **********"
  puts page.html
end
```

To print the last 500 lines of `log/test.log`, also add:


```ruby
After do |scenario|
  if scenario.failed?
    puts "********* PAGE: **********"
    puts page.html

    puts "********* TEST LOG: **********"
    puts `tail -n 500 log/test.log`
  end
end
```
