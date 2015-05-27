When you are using only one role with Devise you may want to change the sign in and sign out routes to <b>/login</b> and <b>/logout</b> (instead of `/users/sign_in` and `/users/sign_out`).
当将Devise用于一个角色，可能想改变登录和退出路由到/login和/logout（而不是`/users/sign_in`和`/users/sign_out`）。

This does not work by default because Devise inspects the URL to find which scope you are accessing. 
默认下这是不行的，因为Devise期待地址来寻找那个要访问的作用域。
So when it sees `/users/login`, it knows the scope is `user`, however, when you access `/login`, Devise cannot know which scope it should use. 
所以，当它看到`/users/login`,它知道作用域是`user`；可是你访问`/login`，Devise就不知道它可以使用哪个作用域了。
Luckily, Devise provides a mechanism to specify a default scope, allowing us to have short URLs.
幸运的是，Devise提供了一个机制来指定默认作用域，允许我们使用短地址。

## Steps for Rails 3.0.0 forward

All you need to do is to specify in your routes the `devise_scope` being accessed in that URL:
你需要做的事情是在路由中指定`devise_scope`，以该地址访问：

```ruby
devise_scope :user do
  get "/login" => "devise/sessions#new"
end
```

Since `devise_scope` is aliased to `as`, this is equivalent:
因为`devise_scope`是`as`的别名，这等价于：

```ruby
as :user do
  get "/login" => "devise/sessions#new"
end
```

Similarly for `sign_out`:
类似的`sign_out`：

```ruby
devise_scope :user do
  delete "/logout" => "devise/sessions#destroy"
end
```

Note that you can skip all sessions routes and define only your own using the skip option as below:
注意可以用下面的skip选项跳过所有的会话路由，只定义你要用的

```ruby
  devise_for :users, :skip => [:sessions]
  as :user do
    get 'signin' => 'devise/sessions#new', :as => :new_user_session
    post 'signin' => 'devise/sessions#create', :as => :user_session
    delete 'signout' => 'devise/sessions#destroy', :as => :destroy_user_session
  end
```

This way `:authenticate_user!` and other helpers will be redirecting the user to the proper custom pages you defined.
这样`:authenticate_user!`和其他辅助器将重定向用户到你定义的定制页面中。

Note that if you are making use of the `:sign_out_via` configuration option, then the `signout` action above may cause errors.
注意，如果正在使用`:sign_out_via`配置选项，那么`signout`动作将会出现错误。
 You can duplicate the default behavior (which changes from `delete` to `get` based on `:sign_out_via`) by specifying:
可以通过指定下面代码复制默认表现（基于`:sign_out_via`从`delete`改到`get`）

```ruby
  devise_for :users, :skip => [:sessions]
  as :user do
    get 'signin' => 'devise/sessions#new', :as => :new_user_session
    post 'signin' => 'devise/sessions#create', :as => :user_session
    match 'signout' => 'devise/sessions#destroy', :as => :destroy_user_session,
      :via => Devise.mappings[:user].sign_out_via
  end
```

## Another simple way to do the same thing  另一个简单方法

The `devise_for` method has a lot of optional parameters to make things easier.
`devise_for`方法有许多可选参数来简化事情。
If you just want to remove the `users` namespace before all the routes and rename `sign_in`, `sign_out` to `login`, `logout`.
如果只想将`users`命名空间在所有路由之前移除，并将`sign_in`和`sign_out`重命名为`login`和`logout`。
Try this way:
试着这样：

```ruby
devise_for :users, :path => '', :path_names => {:sign_in => 'login', :sign_out => 'logout'}
```

if you want to use multiple Devise controllers (like passwords) you should still use the first way.
如果你想使用多个Devise控制器（比如密码），应该使用第一个方法。
