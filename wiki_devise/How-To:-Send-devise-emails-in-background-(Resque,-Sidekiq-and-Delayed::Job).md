Since sending email synchronously is not a good idea, you'll probably want to have Devise enqueuing it's notification emails for background processing.

Although Devise doesn't support this out of the box you can achieve it easily by using the [devise-async](https://github.com/mhfs/devise-async) gem.

To do so, first add it to your Gemfile:

```ruby
gem "devise-async"
```

### For Devise >= 2.1.1
Add `:async` to the list of modules to include in your model's `devise` call.

```ruby
class User < ActiveRecord::Base
  devise :database_authenticatable, :async, :confirmable # etc ...

end
```

### For Devise < 2.1.1
Tell Devise to use the proxy mailer in `config/initializers/devise.rb`:

```ruby
# Configure the class responsible to send e-mails.
config.mailer = "Devise::Async::Proxy"
```

### For all versions
And last but not least, set your queuing backend by creating `config/initializers/devise_async.rb`:

```ruby
# Supported options: :resque, :sidekiq, :delayed_job
Devise::Async.backend = :resque
```

Your notification emails should now be gracefully enqueued for background processing.