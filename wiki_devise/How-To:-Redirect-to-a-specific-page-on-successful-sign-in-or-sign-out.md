Devise will by default redirect to the root_path. However, if you defined a `user_root_path` for your user model (`admin_root_path` for an admin model and so on), Devise will use it instead:

```ruby
match '/welcome' => "welcome#index", as: :user_root

Using  Rails 4
get '/welcome' => "welcome#index", as: :user_root
```


If you want more fine grained control, you can simply override `after_sign_in_path_for`:

```ruby
def after_sign_in_path_for(resource)
  stored_location_for(resource) || welcome_path
end
```
To make the above work, the root path obviously needs to be publicly visible!

If what you really want is to change the way `/` redirects for logged-in vs. unauthenticated users, you can use the `authenticated` route helper:

```ruby
  authenticated :user do
    root to: 'welcome#index', as: :authenticated_root
  end

  root to: 'landing#index'
```

#### Route Naming Note
As of Rails 4+, two routes with the same **name** will raise an exception at app load time. In the above example, it is important to note that the two routes respond to the same URL but they have two different route names (`authenticated_root_path` and `root_path`).

### After signing out

This works a lot like the above, except you use the method:

```ruby
def after_sign_out_path_for(resource_or_scope)
  # logic here
end
```