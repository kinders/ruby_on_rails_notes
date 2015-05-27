These are instructions for allowing you to use resource test doubles instead of actual ActiveRecord objects for testing controllers where Devise interactions are important. This means there's even less reliance on the database for controller specs, and that means faster tests!

This approach certainly needs some evolution, but it's a start.

To stub out your user for an action that expects an authenticated user, you'll want some code like this (once you've got `Devise::TestHelpers` loaded):

```ruby
  user = double('user')
  allow(request.env['warden']).to receive(:authenticate!) { user }
  allow(controller).to receive(:current_user) { user }
```

And for behaviours where the user is not signed in:

```ruby
  request.env['warden'].stub(:authenticate!).
    and_throw(:warden, {:scope => :user})
```

If you're paying attention, you'll see there's the scope for the given resource - adapt that as necessary (and it's especially important if you're using more than one resource scope for Devise in your app).

At the time of writing, only the latest version of `rspec-mocks` supports throw parameters, so make sure you're using 2.8.0 or newer of `rspec` and/or `rspec-rails`.

```ruby
  gem 'rspec-rails', '~> 2.8.0'
```

As a more fleshed out example, I have the following in a file I've added to my app at `spec/support/controller_helpers.rb`:

```ruby
  module ControllerHelpers
    def sign_in(user = double('user'))
      if user.nil?
        allow(request.env['warden']).to receive(:authenticate!).and_throw(:warden, {:scope => :user})
        allow(controller).to receive(:current_user).and_return(nil)
      else
        allow(request.env['warden']).to receive(:authenticate!).and_return(user)
        allow(controller).to receive(:current_user).and_return(user)
      end
    end
  end
  
  RSpec.configure do |config|
    config.include Devise::TestHelpers, :type => :controller
    config.include ControllerHelpers, :type => :controller
  end
```

And then in controller examples we can just call `sign_in` to sign in as a user, or `sign_in nil` for examples that have no user signed in. Here's two quick examples:

```ruby
  it "blocks unauthenticated access" do
    sign_in nil
    
    get :index
    
    response.should redirect_to(new_user_session_path)
  end
  
  it "allows authenticated access" do
    sign_in
    
    get :index
    
    response.should be_success
  end
```