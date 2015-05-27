Devise provides a number of useful functions for use in the config/routes.rb file. Check the source code! Some of the most useful are "authenticate", "authenticated", and "unauthenticated". 

With "authenticate", it is possible to make resources and routes that will ask for authentication before they can be accessed. Devise will automatically take the user to the page they were attempting to access, once they are logged in. A simple example is here:

```ruby
authenticate :user do
  resources :fjords, only: [:new, :create, :edit, :update, :destroy]
end
resources :fjords, only: [:index, :show]
```

Replacing "authenticate" with "authenticated" in the above code causes the resource to simply be unavailable if the user is not authenticated. Replacing "authenticate" with "unauthenticated" allows access to the first list of actions on the controller only if the user is NOT authenticated!
It's also possible to alter the URL scheme for particular actions. Imagine you had a very simple site with only admin logins. The following routing file segment would put all the fjord-editing tools on the /admin path:

```ruby
authenticate :user do
  scope "/admin" do
    resources :fjords, only: [:new, :create, :edit, :update, :destroy]
  end
end
resources :fjords, only: [:index, :show]
```

The above are both route-aspected ways to restrict access to particular resources. Most commonly documented is using the controller to restrict access to its own actions, which is an alternate but not necessarily better way to design an application. To do this, put the following line near the top of the relevant controller, inside the class file but before the action definitions:  

```ruby
before_filter :authenticate_user!, :except => [:show, :index]
```

Testing route-aspected authentication can be a bit tricky. Another page on this wiki has a good solution:
[How-To: Controllers tests with Rails 3 and rspec](https://github.com/plataformatec/devise/wiki/How-To:-Test-controllers-with-Rails-3-and-4-(and-RSpec))

Relevant to the sort of thing that this wiki article is about:
At the time of writing, [Devise (Un)Authenticated Routes Are Broken in Rails](https://github.com/plataformatec/devise/issues/2393) because
[Rails 4 router doesn't support identically named routes with conditional logic](https://github.com/plataformatec/devise/issues/2393).

If you want your own URL structure, see: [How To: Require authentication for all pages](https://github.com/plataformatec/devise/wiki/How-To:-Require-authentication-for-all-pages)