## redirect back to current page after signin?
登录之后重定向回当前页面

See [[How to: redirect to a specific page on successful sign in and sign out]].
参见《成功登录或登出之后重定向到特定页面》

## redirect back to current page after oauth signin?
oauth登录之后重定向回当前页面

This is pretty straight forward, for an oauth sign in, `request.env['omniauth.origin']` is automatically set.
这是相当直接的，对于第三方登录，`request.env['omniauth.origin']`是自动设置的。
You can also fall back to whatever you'd like:
也可以退回你想要的页面

```ruby
class ApplicationController < ActionController::Base
  def after_sign_in_path_for(resource)
    request.env['omniauth.origin'] || stored_location_for(resource) || root_path
  end
end
```

## redirect back to current page **without** oauth signin?
无需oauth登录之后重定向回当前页面

```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery   
          
  def after_sign_in_path_for(resource)
    sign_in_url = new_user_session_url
    if request.referer == sign_in_url
      super
    else
      stored_location_for(resource) || request.referer || root_path
    end
  end
end
```

If you don't put stored_location_for before request.referer you'll get some weird behaviour and sometimes, you won't get to the stored location.
如果不把stored_location_for放在referer.referer之前，结果会很怪异，你不会得到存储的位置。

## Preventing redirect loops 阻止重定向循环

Because the code for `after_sign_in_path_for` above only checks if `request.referer == sign_in_url`, these methods (which call `after_sign_in_path_for`) will also have to be overridden (else you will encounter redirect loops):
因为上面`after_sign_in_path_for`的代码只会检查`request.referer == sign_in_url`，这些方法（调用了`after_sign_in_path_for`的）也必须要改写（否则会出现重定向循环）：

* ```Devise::PasswordsController#after_resetting_password_path_for```
* ```Devise::RegistrationsController#after_sign_up_path_for```
* ```Devise::RegistrationsController#after_update_path_for```

This can be done like so:
类似于：

```ruby
# routes.rb
devise_for :users, controllers: { registrations: 'users/registrations', passwords: 'users/passwords' }

# users/registrations_controller.rb
class Users::RegistrationsController < Devise::RegistrationsController
  protected
    def after_sign_up_path_for(resource)
      signed_in_root_path(resource)
    end

    def after_update_path_for(resource)
      signed_in_root_path(resource)
    end
end

# users/passwords_controller.rb
class Users::PasswordsController < Devise::PasswordsController
  protected
    def after_resetting_password_path_for(resource)
      signed_in_root_path(resource)
    end
end
```
