If you're using the `rails-api` gem, you'll need to manually include the controller methods for CanCan

```ruby
class ApplicationController < ActionController::API
  include CanCan::ControllerAdditions
end
```

You can then use CanCan as usual.