# vim: set ft=markdown :#
# 开启第三方认、退出登录、和记忆
By default, Devise doesn't add a `sign_out` route when using Omniauthable.
使用Omniauthable的默认情况下，Devise不会添加`sign_out`路由。
As your user have logged in through a third-party provider, it will not be able to log out unless you add the following code and adds a link to the sign out action.
用户通过第三方认证登录之后，如果没有添加下列代码和链接给退出登录的动作，就不能登出。

```ruby
devise_scope :user do
   get 'sign_out', :to => 'devise/sessions#destroy', :as => :destroy_user_session
end
```

However, as we're only using session, closing the browser will be enough to sign out.
不过，如果我们使用会话，关闭浏览器就会退出登录了。

You may also notice that rememberable doesn't work because we don't send the rememberable check_in on login.
To enforce rememberable usage, you can add this function call to your omniauth callback controller (when `@user` is the resource):
你也注意到rememberable模块不能运行，因为没有登录时没有发送记住状态的check_in。
要强制使用rememberable，可以添加这个函数调用到omniauth回调控制器中（@user是资源时）：

```ruby
remember_me(@user)
```

You should include module `Devise::Controllers::Rememberable` on your controller to use it and ensure a password is always set or have a `remember_token` column in your model or implement your own `rememberable_value` in the model with custom logic.
可以包含`Devise::Controllers::Rememberable`模块在控制器中来使用它，确保密码总是设置完毕，或者在模型中有`remember_token`列，或者在模型中用自定义的逻辑实现自己`remember_me`功能。

This way, the logged in status will persist between sessions.
这样，登录状态会在会话之间持续。
We don't recommend doing this if you don't have a `sign_out` action (because people will not be able to log out, even if they close the browser).
如果没有`登出`动作，人们就不能退出登录，即使关闭了浏览器，所以我们不推荐这么做。

