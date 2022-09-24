Shows exception uuids and links to them on error pages for easier support

![example](assets/example.png)

Install
=======

```Bash
gem install rollbar-user_informer
```

Details
=====
* Adds rack middleware `Rollbar::UserInformer::Middleware` that inserts a link to Rollbar in error pages.
* Allows to choose full (for instance with link to rollbar) and short (just an id) message using `Rollbar::UserInformer.user_information_show_full` proc 
* Allows customization of error message with `Rollbar::UserInformer.user_information` and `Rollbar::UserInformer.user_information_full` (replaces `{{error_uuid}}` 
with the error uuid from Rollbar.)
* Allows customizing error page placeholder `Rollbar::UserInformer.user_information_placeholder`, must match 
what is on the error page (default is `<!-- ROLLBAR ERROR -->`).

Usage
=====
```ruby
# Gemfile
gem 'rollbar-user_informer'

# config/initializers/rollbar.rb
Rollbar.configure do |config|
...
end

Rollbar::UserInformer.user_information = <<~HTML
  <p>Incident ID: {{error_uuid}}</p>
HTML

Rollbar::UserInformer.user_information_full = <<~HTML
  <p><a href="#{Rollbar.notifier.configuration.web_base}/instance/uuid?uuid={{error_uuid}}">
    View error {{error_uuid}} on Rollbar
  </a></p>
HTML

Rollbar::UserInformer.user_information_show_full = lambda do |env|
  request = ActionDispatch::Request.new(env)
  request.cookie_jar.signed[:show_rollbar_link].to_b
end
```

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base

  ...
    
  before_action :enable_rollbar_link
  
  def enable_rollbar_link
    if current_user && current_user.has_role?(:admin)
      cookies.signed.permanent["show_rollbar_link"] = true
    end
  end

  ...
  
end

```

```html
# public/500.html
<body>
  <div class="dialog">
    <h1>We're sorry, but something went wrong.</h1>
  </div>
  <p>
    If you are the application owner check the logs for more information.
    <!-- ROLLBAR ERROR -->
  </p>
</body> 
```

Author
======
[Ryan Gurney](https://github.com/ragurney)<br/>
ryan.a.gurney@gmail.com<br/>
License: MIT<br/>
[![Build Status](https://travis-ci.org/ragurney/rollbar-user_informer.svg?branch=master)](https://travis-ci.org/ragurney/rollbar-user_informer)
