Currently, Devise's Omniauthable module does not work with multiple models. No need to worry though, as the Omniauthable module is but a simple wrapper around OmniAuth.

So to allow OmniAuth authentication for multiple models:

1. Remove the `:omniauthable` argument from your models.

2. Move the OmniAuth configuration out of Devise's configuration file and into a new file. So, instead of having this in `devise.rb`:

  ```ruby
  Devise.setup do |config|
    config.omniauth :twitter, ENV['TWITTER_KEY'], ENV['TWITTER_SECRET']
  end
  ```

  You now have this in your new file `omniauth.rb`:

  ```ruby
  Rails.application.config.middleware.use OmniAuth::Builder do
    provider :twitter, ENV['TWITTER_KEY'], ENV['TWITTER_SECRET']
  end
  ```

3. Create your own OmniAuth routes. Here's an example of a possible route, given you have an `authentications_controller.rb`:

  ```ruby
  get "/auth/:action/callback", :to => "authentications", :constraints => { :action => /twitter|google/ }
  ```

  Or if you want all OmniAuth authentications to be directed to the same `create` action, independently of the provider used:

  ```ruby
  get "/auth/:provider/callback" => "autentications#create"
  ```

4. Setup OmniAuth's `on_failure` hook. You can do this inside OmniAuth's configuration block:

  ```ruby
  Rails.application.config.middleware.use OmniAuth::Builder do
    on_failure { |env| AuthenticationsController.action(:failure).call(env) }
  end
  ```

  Or outside of it:

  ```ruby
  OmniAuth.config.on_failure = Proc.new { |env| AuthenticationsController.action(:failure).call(env) }
  ```

  [Devise implements this second option][1], but the first option is simpler since OmniAuth is already being configured on the `omniauth.rb` file.

    With this set up, all authentication failures (such as a user cancelling logging in) will be redirected to the `failure` action in the `authentications_controller.rb`.

5. Handle OmniAuth failures. Since errors in this library aren't properly standardized, the only way to handle them is with lots of conditionals, [like Devise is doing][2]. Simply copy the relevant code and you should be good to go.

[The Railscast on Simple OmniAuth][3] should also help setting this up.

[1]: https://github.com/plataformatec/devise/blob/master/lib/devise/omniauth.rb
[2]: https://github.com/plataformatec/devise/blob/master/app/controllers/devise/omniauth_callbacks_controller.rb
[3]: http://railscasts.com/episodes/241-simple-omniauth