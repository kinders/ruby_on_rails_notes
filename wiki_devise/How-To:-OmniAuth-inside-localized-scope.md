# 在本地作用域中使用OmniAuth

[gurix](https://github.com/gurix) provided the following solution in response to [issue #272](https://github.com/plataformatec/devise/issues/2813)
提供了下面的解决方案来回应问题#272.

First we need a controller inside the locale scope that sets us the current locale in the session.
首先需要一个在本地化作用域内的控制器，在会话中设置当前本地。

`routes.rb`:

```RUBY
Rails.application.routes.draw do
  # We need to define devise_for just omniauth_callbacks:auth_callbacks otherwise it does not work with scoped locales
  # 需要定义omniauth_callbacks:auth_callbacks的devise_for，否则它不会在本地作用域下运行：
  # see https://github.com/plataformatec/devise/issues/2813

  devise_for :users, skip: [:session, :password, :registration, :confirmation], controllers: { omniauth_callbacks: 'omniauth_callbacks' }

  scope '(:locale)' do
    # We define here a route inside the locale thats just saves the current locale in the session
    # 这里在本地化里面定义一个路由，只在会话中保存当前本地化
    get 'omniauth/:provider' => 'omniauth#localized', as: :localized_omniauth

    devise_for :users, skip: :omniauth_callbacks, controllers: { passwords: 'passwords', registrations: 'registrations' }
  end
end
```

`omniauth_controller.rb`:
在omniauth控制器中：

```RUBY
class OmniauthController < ApplicationController
  def localized
    # Just save the current locale in the session and redirect to the unscoped path as before
    # 只在会话保存当前本地区，并像以前那样重定向到非作用域路径。
    session[:omniauth_login_locale] = I18n.locale
    redirect_to user_omniauth_authorize_path(params[:provider])
  end
end
```

Now we can force the locle inside the `omniauth_callbacks_controller.rb`:
现在可以强制`omniauth_callbacks_controller.rb`本地化了：

```RUBY
class OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def twitter
    handle_redirect('devise.twitter_uid', 'Twitter')
  end

  def facebook
    handle_redirect('devise.facebook_data', 'Facebook')
  end

  private

  def handle_redirect(_session_variable, kind)
    # Use the session locale set earlier; use the default if it isn't available.
    # 使用早前的会话本地设置，如果不行就使用默认的。
    I18n.locale = session[:omniauth_login_locale] || I18n.default_locale
    sign_in_and_redirect user, event: :authentication
    set_flash_message(:notice, :success, kind: kind) if is_navigational_format?
  end

  def user
    User.find_for_oauth(env['omniauth.auth'], current_user)
  end
end
```

That's all, every time you login via `link_to t('.sign_up_with_twitter'), localized_omniauth_path(:twitter)`, or whatever, it forces the intended locale. 
就行了。每次通过使用`link_to t('.sign_up_with_twitter'), localized_omniauth_path(:twitter)`登录，它都会转到本地页面中。

Also as gist https://gist.github.com/gurix/4ed589b5551661c1536a
