# Rails Tips

Surprising or less know things from the Ruby on Rails universe

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
