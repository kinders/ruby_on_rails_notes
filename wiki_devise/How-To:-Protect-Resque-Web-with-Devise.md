Resque is an excellent plugin for creating background jobs on multiple queues and processing them later. 

[resque/resque](https://github.com/resque/resque) 

It ships with an extremely simple to use Sinatra app for viewing and managing your worker queues. However, this Sinatra app (called resque-web) gets mounted unprotected by Devise initially. 

```ruby
#routes.rb
...
mount Resque::Server.new, :at => '/resque'
...
```

Luckily, there is a simple way to add Devise authentication to the resque-web front end app.

```ruby
#routes.rb
...
  devise_for :admin_users, ActiveAdmin::Devise.config
  authenticate :admin_user do #replace admin_user(s) with whatever model your users are stored in.
    mount Resque::Server.new, :at => "/jobs"
  end
...
```

That's it! If an un-authenticated user attempts to visit `/jobs` they'll get redirected to your devise login page, after logging in devise will redirect them back to the `/jobs` URL. (Or whatever URL they were attempting to access, `/jobs/schedule` etc.) 

