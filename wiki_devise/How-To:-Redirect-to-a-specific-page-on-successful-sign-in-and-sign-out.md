You can override the default behaviour by creating an `after_sign_in_path_for` [[RDoc](http://rubydoc.info/github/plataformatec/devise/master/Devise/Controllers/Helpers:after_sign_in_path_for)] method in your ApplicationController and have it return the path for the page you want:

```ruby
def after_sign_in_path_for(resource)
  current_user_path
end
```

There exists a similar method for sign out; `after_sign_out_path_for` [[RDoc](http://rubydoc.info/github/plataformatec/devise/master/Devise/Controllers/Helpers:after_sign_out_path_for)]

## Keeping user on the same page after signing out

```ruby
def after_sign_out_path_for(resource_or_scope)
  request.referrer
end
```
