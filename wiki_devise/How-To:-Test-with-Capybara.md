Acceptance tests for your application may require that a test user be logged in. To do this in your application you can either sign in the user using capybara by visiting the sign in url and 
entering valid credentials, or you can stub the logged in user.

Signing in via Capybara is perhaps the most 'correct' way since we want our tests to be as close to real world conditions as possible, however if you have more than a few tests, these few extra actions for each logged in test can be quite time consuming. The alternative is to use warden's built in stubbing actions to make your application think that a user is signed in but without all of the overhead of actually signing them in.

To do this, you'll need to include warden test helpers and turn on test mode

```ruby
include Warden::Test::Helpers
Warden.test_mode!
```

Then you can make a call to the warden helper `login_as` with a user resource and specifying the `:scope => :user` to 'log in' a test user.

```ruby
user = FactoryGirl.create(:user)
login_as(user, :scope => :user)
```
You will then need to create a corresponding user factory in your factories file (e.g. `spec/factories.rb`, `test/factories.rb`). This will look something like this:

```ruby
FactoryGirl.define do
  factory :user do
    email 'test@example.com'
    password 'f4k3p455w0rd'

    # if needed
    # is_active true
  end
end
```
If you have added any fields to your User model, you will need to add these here. For more details see the [FactoryGirl docs](https://github.com/thoughtbot/factory_girl/wiki/Usage).

If you are using devise's confirmable module, you will need to ensure that the newly created user also has the 'confirmed_at' field filled in: 

```ruby
user = FactoryGirl.create(:user)
user.confirmed_at = Time.now
user.save
```

To make sure this works correctly you will need to reset warden after each test. You can do this by calling

```
Warden.test_reset! 
```

If for some reason you need to log out a logged in test user, you can use Warden's `logout` helper.

```ruby
logout(:user)
```

Now you should be good to go. 

If you're wondering why we can't just use Devise's built in `sign_in` and `sign_out` methods, it's because these require direct access to the request object which is not available while using Capybara. To bundle the functionality of both methods together you can create a helper method. 

## Capybara-Webkit
If you have trouble using Warden's login_as method with the **capybara-webkit** driver, try setting run_callbacks to false in the login_as options struct
```ruby
user = FactoryGirl.create(:user)
login_as(user, :scope => :user, :run_callbacks => false)
```

## If your login works using Capybara/Rack::Test, but not in Capybara/Selenium
make sure to set transactional fixtures to false in your Rspec spec_helper.rb file
```ruby
RSpec.configure do |config|
    config.use_transactional_fixtures = false
end
```
See answers here for details: http://stackoverflow.com/questions/6154687/rails-integration-test-with-selenium-as-webdriver-cant-sign-in

See also: http://stackoverflow.com/a/12330557/3137294

## Capybara and Poltergeist
login_as may not work with Poltergeist when js is enabled on your test scenarios. To fix this, add [this file](https://github.com/railscasts/391-testing-javascript-with-phantomjs/blob/master/checkout-after/spec/support/share_db_connection.rb) to your app.

See [this SO question](http://stackoverflow.com/questions/18623661/why-is-capybara-discarding-my-session-after-one-event) for more explanation

Good luck and happy testing. 
[Blog post with more details](http://schneems.com/post/15948562424/speed-up-capybara-tests-with-devise)