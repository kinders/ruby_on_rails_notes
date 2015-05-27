# 使用recaptcha
**WARNING: The first draft of this page was done by a total n00b. There wasn't any help here for Recaptcha before, so he hopes this does help some people, but other more experienced should clean this up at a later date.**

1. Get your keys from [Google/Recaptcha](http://www.google.com/recaptcha)
2. Install the Recaptcha gem [found here](https://github.com/ambethia/recaptcha) (note: for Rails 3 do `gem 'recaptcha', :require => 'recaptcha/rails'`)
3. Add `<%= recaptcha_tags %>` on your New Registration view (you must have generated Devise views) the line before your submit button.
4. Create/Update your RegistrationsController - see the "controllers" section in the README to understand how to set up your own devise controllers, and don't forget to update the routes once you do.  Remember that "devise" expects you to use flash[:notice] and flash[:alert] and not flash[:error].  Include the following (the first is for a clean Devise install, and the second is for if you use OmniAuth like me):

### Clean

```ruby
    class RegistrationsController < Devise::RegistrationsController
      def create
        if verify_recaptcha
          super
        else
          build_resource(sign_up_params)
          clean_up_passwords(resource)
          flash.now[:alert] = "There was an error with the recaptcha code below. Please re-enter the code."      
          flash.delete :recaptcha_error
          render :new
        end
      end
    end
```

5. In Rails 3.x you may need to add:

```ruby
# ./config/application.rb
require 'net/http'
```
to support the Recaptcha gem.

6. With Ruby 1.9.x, you need to add:

```ruby
# ./config/routes.rb
devise_for :users, controllers: { registrations: 'registrations' }
```

With Ruby 1.8.x:

```ruby
# ./config/routes.rb
devise_for :users, :controllers => { :registrations => 'registrations' }
```

### OmniAuth:

```ruby
  class RegistrationsController < Devise::RegistrationsController
  
    def create
      if session[:omniauth] == nil #OmniAuth
        if verify_recaptcha
          super
          session[:omniauth] = nil unless @user.new_record? #OmniAuth
        else
          build_resource
          clean_up_passwords(resource)
          flash[:alert] = "There was an error with the recaptcha code below. Please re-enter the code."
          #use render :new for 2.x version of devise
          render_with_scope :new 
        end
      else
        super
        session[:omniauth] = nil unless @user.new_record? #OmniAuth
      end
    end
```

## Alternate implementation

**Warning: I'm not sure I'm any more qualified on this than the OP, but I preferred this setup in my own app.**

**Note (advice by reader): If the sign-up is by email as primary key, giving an error that the email is already taken (regardless of recaptcha result) means that it could be used for email harvesting. **

The benefit to this method is that we can add any recaptcha errors to the rest of the form errors instead of an either/or scenario as would happen with the controllers listed above.

Steps 1 - 4 are the same

```ruby
class Users::RegistrationsController < Devise::RegistrationsController
  def create
    if !verify_recaptcha
      flash.delete :recaptcha_error
      build_resource(sign_up_params)
      resource.valid?
      resource.errors.add(:base, "There was an error with the recaptcha code below. Please re-enter the code.")
      clean_up_passwords(resource)
      respond_with_navigational(resource) { render_with_scope :new }
    else
      flash.delete :recaptcha_error
      super
    end
  end
  
  def clean_up_passwords(*args)
    # Delete or comment out this method to prevent the password fields from 
    # repopulating after a failed registration
  end
end
```

The `flash.delete :recaptcha_error` gets rid of the default error from the recaptcha gem, which in my case kept appearing as "incorrect-captcha-sol".

For Devise 2.x change

```
respond_with_navigational(resource) { render_with_scope :new }
```

into

```
respond_with_navigational(resource) { render :new }
```

## Notice! Captcha for Login 

If you want use captcha on your login page, you need to care about a security issue!

By default, Devise has use [`require_no_authentication`](https://github.com/plataformatec/devise/blob/972ac3b5f0d9a0cbc8ff2cbdb0b52044acc5c284/app/controllers/devise_controller.rb#L116) method as before_filter for `Session#new` action.

**That means Devise will auto check login and do login event before you `Session#new` action, so you `verify_recaptcha` check will can't work.**

So you need skip that method like this:

```ruby
class SessionsController < Devise::SessionsController
  skip_before_filter :require_no_authentication, :only => [:new]  

  def create
    if verify_recaptcha
      super
    else
      build_resource
      flash[:error] = "Captcha has wrong, try a again."
      respond_with_navigational(resource) { render :new }
    end    
  end
end
```

More info about this: [#2016](https://github.com/plataformatec/devise/issues/2106)

## Captcha with Devise 2.x
If the above method doesn't work then try
```ruby
  before_filter :check_user_validation, :only=>:create

  def check_user_validation
     #validate user or redirect with this method
  end
```

## Recaptcha::RecaptchaError

This error `getaddrinfo: nodename nor servname provided, or not known`, can be due to misconfigured
initializer (recaptcha.rb) or rails not loading the `net/http` library.
The [Alternate implementation](#alternate-implementation) worked for me after I added it as below.

```ruby
# config/application.rb

require 'net/http'
```
