# vim: set ft=markdown :#
# 第三方认证概览

Since version 1.2, Devise supports integration with [OmniAuth](http://github.com/intridea/omniauth).
到了1.2版，Devise支持集成OmniAuth。
This wiki page will cover the basics to have this integration working using an OAuth provider as example.
这个维基页面将用一个OAuth提供器的例子涵盖实现集成的基本。

Since version 1.5, Devise supports OmniAuth 1.0 forward which will be the version covered by this tutorial.
到了1.5版本，Devise支持OmniAuth 1.0以上，本章都将会涵盖到。

# Before you start 开始之前

Remember that config.omniauth adds omniauth provider middleware to your application.
记住config.omniauth给程序添加了omniauth供应器中间件。
This means you should **not** add this provider middleware again in config/initializers/omniauth.rb as they'll clash with each other and result in always-failing authentication.
这意味着你不应该在config/initializers/omniauth.rb中再次添加这个供应器中间件，因为它们互相冲突并导致认证失败。


# Facebook example 脸书的例子

The first step then is to add an OmniAuth gem to your application. This can be done in our Gemfile:
第一步，添加OmniAuth软件包到程序中。
在Gemfile中：

```ruby
gem 'omniauth-facebook'
```

Here we'll use Facebook as an example, but you are free to use whatever and as many OmniAuth gems as you'd like.
这里使用脸书为例子，你可以使用任何喜欢的OmniAuth软件包。
Generally, the gem name is "omniauth-provider" where provider can be "facebook" or "twitter", for example.
一般，gem的名称是“omniauth-provider”，provider可以是脸书或推特等。
For a full list of these providers, please check [OmniAuth's list of strategies](https://github.com/intridea/omniauth/wiki/List-of-Strategies).
全部providers的列表参见网页。


Next up, you should add the columns "provider" (string) and "uid" (string) to your User model.
下一步，应该将"provider"和"uid"两栏添加到User模型中。

```rails
rails g migration AddColumnsToUsers provider uid
rake db:migrate
```

Next, you need to declare the provider in your config/initializers/devise.rb:
下一步，需要在config/initializers/devise.rb中声明提供者。

```ruby
config.omniauth :facebook, "APP_ID", "APP_SECRET"
```

and replace `APP_ID` and `APP_SECRET` with your app id and secret.
将`APP_ID`和`APP_SECRET`替换成你的。

To alter the permissions or scopes requested, check the [omniauth-facebook](https://github.com/mkdynamic/omniauth-facebook) gem's README.
要改变权限或限定请求，参见软件包的自述。

After configuring your strategy, you need to make your model (e.g. app/models/user.rb) omniauthable:
配置策略之后，需要开启模型的omniauth功能。

```ruby
devise :omniauthable, :omniauth_providers => [:facebook]
```

_Note: If you're running a rails server, you'll need to restart it to recognize the change in the Devise initializer or adding :omniauthable to your User model will create an error._
注意：如果运行rails服务器，需要重启来识别这些对启动器或模型功能的更改，否则会出现错误。

Currently, Devise only allows one model to be omniauthable.
现在，Devise只允许一个模型开启第三方认证。
If you want to use OmniAuth with multiple models, check out [OmniAuth with multiple models](https://github.com/plataformatec/devise/wiki/OmniAuth-with-multiple-models).
如果需要多个模型开启该功能，参见维基文件。

After making a model named `User` omniauthable and if `devise_for :users` was already added to your config/routes.rb, Devise will create the following url methods:
在模型User开启OmniAuth之后，如果`devise_for :users`已经添加到config/routes.rb，Devise将会创建下面的地址辅助方法：

* user_omniauth_authorize_path(provider)
* user_omniauth_callback_path(provider)

Note that devise does not create `*_url` methods. While you will never use the callback helper above directly, you only need to add the first one to your layouts in order to provide facebook authentication:
注意devise不会创建`*_url`方法。
因为不会直接使用上面的回调辅助器，只需要添加第一个到布局钟，以便提供脸书认证。

```ruby
<%= link_to "Sign in with Facebook", user_omniauth_authorize_path(:facebook) %>
```

_Note: The symbol passed to the_ user_omniauth_authorize_path _method matches the symbol of the provider passed to Devise's config block. Please check that omniauth-* gem's README to know which symbol you should pass._
注意：传给`*user_omniauth_authorize_path`方法的符号匹配传给Devise的配置代码块的供应器的符号。
请参考`omniauth-*`软件包的自述文件，看你可以传递哪个符号。

By clicking on the above link, the user will be redirected to Facebook.
点击上面的链接，用户将被重定向到脸书。
(If this link doesn't exist, try restarting the server.) 
如果链接不存在，请重启服务器。
After inserting their credentials, they will be redirected back to your application's callback method.
完成认证之后，将被重定向回程序的回调方法。
To implement a callback, the first step is to go back to our config/routes.rb file and tell Devise in which controller we will implement Omniauth callbacks:
实现回调的第一步是回到config/routes.rb中告诉Devise哪个控制器实现Omniauth回调：

```ruby
devise_for :users, :controllers => { :omniauth_callbacks => "users/omniauth_callbacks" }
```

Now we just add the file "app/controllers/users/omniauth_callbacks_controller.rb":
现在只需要添加"app/controllers/omniauth_callbacks_controller.rb"：

```ruby
class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
end
```

The callback should be implemented as an action with the same name as the provider. 
回调将被实现为一个和供应器一样名称的动作。
Here is an example action for our facebook provider that we could add to our controller:
这是一个例子动作，添加到控制器中的脸书供应器：

```ruby
class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def facebook
    # You need to implement the method below in your model (e.g. app/models/user.rb)
    @user = User.from_omniauth(request.env["omniauth.auth"])

    if @user.persisted?
      sign_in_and_redirect @user, :event => :authentication #this will throw if @user is not activated
      set_flash_message(:notice, :success, :kind => "Facebook") if is_navigational_format?
    else
      session["devise.facebook_data"] = request.env["omniauth.auth"]
      redirect_to new_user_registration_url
    end
  end
end
```

This action has a few aspects worth describing:
这个动作有一些方面值得描述：

1. All information retrieved from Facebook by OmniAuth is available as a hash at `request.env["omniauth.auth"]`. 
所有OmniAuth从脸书的信息传回的信息在一个`request.env["omniauth.auth"]`散列中。
Check the [OmniAuth docs](https://github.com/intridea/omniauth/wiki/Auth-Hash-Schema) and each [omniauth-facebook](https://github.com/mkdynamic/omniauth-facebook#auth-hash) gem's README to know which information is being returned.
查看omniauth文档，以及每个gem的自述文件，了解哪些信息是返回的。

2. When a valid user is found, they can be signed in with one of two Devise methods: `sign_in` or `sign_in_and_redirect`. 
一个有效用户被找到时，他们可以通过`sign_in`或`sign_in_and_redirect`方法中的一个来登录。
Passing `:event => :authentication` is optional. 
可以传入`:event => :authentication`。
You should only do so if you wish to use [Warden callbacks](http://stackoverflow.com/a/13389324/1160916).
如果想使用“监管回调”，可以这么做。

3. A flash message can also be set using one of Devise's default messages, but that is up to you.
也可以设置使用Devise的默认信息作为闪存信息，但那取决于你。

4.  In case the user is not persisted, we store the OmniAuth data in the session.
用户没有继续的情况下，将OmniAuth数据存储在会话中。
Notice we store this data using "devise." as key namespace.
注意使用"devise."作为键的命名空间来存储这个数据。
This is useful because Devise removes all the data starting with "devise." from the session whenever a user signs in, so we get automatic session clean up.
因为devise从会话中移除了所有以"devise."开头的数据，不管用户是否登录，因此会自动清除会话，这是很有用的。
At the end, we redirect the user back to our registration form.
最后，重定向用户回到注册表单。

After the controller is defined, we need to implement the `from_omniauth` method in our model (e.g. app/models/user.rb):
定义控制器之后，需要在模型中实现`from_omniauth`方法。

```ruby
def self.from_omniauth(auth)
  where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
    user.email = auth.info.email
    user.password = Devise.friendly_token[0,20]
    user.name = auth.info.name   # assuming the user model has a name
    user.image = auth.info.image # assuming the user model has an image
  end
end
```

This method tries to find an existing user by the `provider` and `uid` fields.
这个方法尝试通过`provider`和`uid`字段找到一个已存的用户。
If no user is found, a new one is created with a random password and some extra information.
如果没有找到用户，新用户将用一个随机密码和另外的信息来创建。
Note that the [`first_or_create` method](http://apidock.com/rails/v3.2.1/ActiveRecord/Relation/first_or_create) automatically sets the `provider` and `uid` fields when creating a new user.
注意创建新用户时，`first_or_create`方法会自动设置`provider`和`uid`字段。
 
Notice that Devise's RegistrationsController by default calls "User.new_with_session" before building a resource.
注意Devise的注册控制器默认情况下会在构建资源之前调用`User.new_with_session`。
This means that, if we need to copy data from session whenever a user is initialized before sign up, we just need to implement `new_with_session` in our model.
这就意味着，如果需要从会话中拷贝数据，只需要在模型中实现`new_with_session`方法即可。
Here is an example that copies the facebook email if available:
这是一个从脸书中复制电邮的例子：

```ruby
class User < ActiveRecord::Base
  def self.new_with_session(params, session)
    super.tap do |user|
      if data = session["devise.facebook_data"] && session["devise.facebook_data"]["extra"]["raw_info"]
        user.email = data["email"] if user.email.blank?
      end
    end
  end
end
```

Finally, if you want to allow your users to cancel sign up with Facebook, you can redirect them to "cancel_user_registration_path".
最后，如果想允许用户取消使用脸书登录，可以将他们重定向到`cancel_user_registration_path`中。
This will remove all session data starting with "devise." and the `new_with_session` hook above will no longer be called.
这回移除以`devise.`开头的所有会话数据，并且不再调用上面的`new_with_session`钩子。

### Logout links  布局链接

config/routes.rb
```ruby
devise_scope :user do
  get 'sign_out', :to => 'devise/sessions#destroy', :as => :destroy_user_session
end
```

And that is all you need! 
After you get your integration working, it's time to write some integration tests:
这就是全部了！
这些集成运行之后，该写些集成测试了。

[https://github.com/intridea/omniauth/wiki/Integration-Testing](https://github.com/intridea/omniauth/wiki/Integration-Testing)

# Using OmniAuth without other authentications  只使用OmniAuth，不使用其他认证

If you are using ONLY omniauth authentication, you need to define a route named new_user_session (if not defined, root will be used).
如果只使用omniauth认证，需要定义一个名称为new_user_session的路由（如果没有定义，将使用根目录）。
Below is an example of such routes (you don't need to include it if you are also using database or other authentication with omniauth):
下面是路由示例（如果也需要使用数据库或其他认证，就不需要这样）：

```ruby
devise_for :users, :controllers => { :omniauth_callbacks => "users/omniauth_callbacks" }

devise_scope :user do
  get 'sign_in', :to => 'devise/sessions#new', :as => :new_user_session
  get 'sign_out', :to => 'devise/sessions#destroy', :as => :destroy_user_session
end
```

In the example above, the sessions controller doesn't need to do anything special.
在上面的示例钟，会话控制器不需要其他东西。
For example, showing a link to the provider authentication will suffice.
比如，显示一个供应器认证的链接就足够了。

Also, if you are not using `:database_authenticatable` you have to define the helper method `new_session_path(scope)` so it can correctly redirect in case of failure:
而且，如果没有使用`:database_authenticatable`，必须定义辅助器方法`new_session_path(scope)`，因此在认证失败时可以正确地重定向：

```ruby
class ApplicationController < ActionController::Base
# ...
  def new_session_path(scope)
    new_user_session_path
  end
end
```

## Troubleshooting  常见问题

### OpenSSL

If you run into an OpenSSL error like this: 
如果出现类似下面的OpenSSL错误：

```ruby
OpenSSL::SSL::SSLError (SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed):
```

Then you need to explicitly tell OmniAuth where to locate your CA certificate file.
需要明确告诉OmniAuth在哪里找到CA认证文件。
Either use the [certified gem](https://github.com/stevegraham/certified) or this method: (depending on the OS you are running on):
或者使用certified软件包 ，或者使用下面这个方法，这取决于运行的操作系统：

```ruby
config.omniauth :facebook, "APP_ID", "APP_SECRET",
  :client_options => {:ssl => {:ca_path => '/etc/ssl/certs'}}
```

On Heroku, the CA file is located at `/usr/lib/ssl/certs/ca-certificates.crt`
在 Heroku，CA文件位于`/usr/lib/ssl/certs/ca-certificates.crt`。

On Engine Yard Cloud servers, the CA file is located at `/etc/ssl/certs/ca-certificates.crt`.
在 Engine Yard Cloud 服务器，CA文件位于`/etc/ssl/certs/ca-certificates.crt`。

These certificates can be set using the `:ca_file` key:
这些认证可以使用`:ca_file`键来设置。

```ruby
config.omniauth :facebook, "APP_ID", "APP_SECRET",
  :client_options => {:ssl => {:ca_file => '/usr/lib/ssl/certs/ca-certificates.crt'}}
```

On OS/X, for development only, it may be easiest just to disable certificate verification because the certificates are stored in the keychain, not the file system:
在 OS/X，对于开发环境，最简单的方式是关闭认证校验，因为认证存储在键链中，不是在文件系统中：

```ruby
require "omniauth-facebook"
OpenSSL::SSL::VERIFY_PEER = OpenSSL::SSL::VERIFY_NONE if Rails.env.development? 
config.omniauth :facebook, "APP_ID", "APP_SECRET"
```

A deeper discussion of this error can be found here: https://github.com/intridea/omniauth/issues/260
更深入的讨论参见omniauth的问题260.

### Cannot load strategy class 不能加载策略类

If for some reason Devise cannot load your strategy class, you can set it explicitly with the `:strategy_class` option:
如果因为不明原因导致不能加载策略类，可以明确地使用选项`:strategy_class`来设置：

```ruby
config.omniauth :facebook, "APP_ID", "APP_SECRET", :strategy_class => OmniAuth::Strategies::Facebook
```

