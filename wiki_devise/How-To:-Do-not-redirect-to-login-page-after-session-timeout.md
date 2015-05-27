By default, devise will redirect to login page after session timeout, but it is weird if you visit a page do not need login.

To solve this problem, you can create custom failure app.

In `config/initializers/devise.rb` configure your custom failure app:

```ruby
require "custom_failure_app"
...
config.warden do |manager|
  manager.failure_app = CustomFailureApp
end
```

And in `lib/custom_failure_app.rb`:

```ruby
class CustomFailureApp < Devise::FailureApp
  def redirect
    store_location!
    message = warden.message || warden_options[:message]
    if message == :timeout     
      redirect_to attempted_path
    else 
      super
    end
  end
end
```
