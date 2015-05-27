You can override the default behaviour by creating an `after_sign_in_path_for` [[RDoc](http://rubydoc.info/github/plataformatec/devise/master/Devise/Controllers/Helpers:after_sign_in_path_for)] method in your `ApplicationController` and have it return the path for the page you want:

```ruby
def after_sign_in_path_for(resource_or_scope)
 current_user_path
end
```

There exists a similar method for sign out; `after_sign_out_path_for` [[RDoc](http://rubydoc.info/github/plataformatec/devise/master/Devise/Controllers/Helpers:after_sign_out_path_for)]

## Keeping user on the same page after signing out

```ruby
def after_sign_out_path_for(resource_or_scope)
  URI.parse(request.referer).path if request.referer
end
```

## After sign up
You need to overwrite this method in your own **RegistrationsController**

```ruby
def after_sign_up_path_for(resource)
  after_sign_in_path_for(resource)
end
```

If the **Confirmed** module is enabled, the method `after_inactive_sign_up_path_for` should be redefined:

```ruby
def after_inactive_sign_up_path_for(resource)
  welcome_index_path
end
```