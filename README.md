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
