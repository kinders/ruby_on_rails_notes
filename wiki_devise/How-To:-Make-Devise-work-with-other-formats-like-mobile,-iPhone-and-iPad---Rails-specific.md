1.Make your app render Apple iOS templates for iOS devices like iPhone, iPad, iPod Touch, etc.
In the Rails `app/controllers/application_controller.rb` add this code:

```ruby
  before_filter :adjust_format_for_iphone
  private
  def adjust_format_for_iphone    
    request.format = :ios if request.env["HTTP_USER_AGENT"] =~ %r{Mobile/.+Safari}
  end
```

2.Set Devise to display views for iOS
In the `config/initializers/devise.rb`, look for the 'config.navigational_formats' and add every other formats you have
`config.navigational_formats = [:"*/*", "*/*", :html, :ios]`

3.Create each view you need for Devise (login form, signup form, ...)
Name your view like 'new.**ios**.erb'
eg: 'app/views/devise/sessions/new.ios.erb' for login form

4.Create an initializer in the folder config/initializers
Add this code into your initializer

```ruby
ActionController::Responder.class_eval do
  alias :to_ios :to_html
end
```
This code has to correspond to your format (ie: :to_ios, :to_ipad, :to_mobile)

5.Add your format to config/initializers/mime_types.rb

```ruby
Mime::Type.register_alias "text/html", :ios
```

6. Note that if you use OAUTH for Facebook you will need to configure it to use facebook's mobile authentication path. This [post](http://www.heathrmoor.com/blog/2012/03/27/devise-omniauth-login-to-mobile-facebook/) explains how.