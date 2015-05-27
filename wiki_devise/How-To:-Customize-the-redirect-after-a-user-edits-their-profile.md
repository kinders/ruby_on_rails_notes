Normally, after a user edits their profile they are redirected to the root_path.
一般，在用户编辑完账户信息之后，将被重定向到根路径。
To have Devise redirect to a custom route after an update, follow these steps:
要让用户更新信息之后重定向到一个自定义路由，请跟着下面步骤修改：

1. Sub-class Devise's registration controller.
   将Devise的注册控制器作为子类。

2. Override the `after_update_path_for(resource)` method.
   改写`after_update_path_for(resource)`方法。

3. Configure Devise's routing.
   配置Devise路由。

Example subclass/override (`registrations_controller.rb`):
子类/覆盖示例：

```ruby
class RegistrationsController < Devise::RegistrationsController

  protected

    def after_update_path_for(resource)
      user_path(resource)
    end
end
```

Example routing config (in `routes.rb`):
路由配置示例：

```ruby
devise_for :users, :controllers => { :registrations => :registrations }
```

The above code, will redirect the user back to the edit page, after the form submits with success.
上面的代码将会在表单成功提交之后，将用户重定向回编辑页面，
Depending on how you have configured your registrations views (if at all) you may need to move them into a different package.
取决于你怎么配置注册器视图，如果想根本些，你需要将它们移动到一个不同的包里。

**Alternatively**, you can define user_root in your config/routes.rb:-
可选的，可以在config/routes.rb中定义用户根路径。

```ruby
devise_for :users do
 get 'users', :to => 'users#show', :as => :user_root # Rails 3
end
```

Where 'users#show' is the controller/action you want to redirect to.
`users#show`是你要重定向的目标控制器动作。

**...or** (though not quite as elegant) you can just add a new match clause to your config/routes.rb:
或者（尽管不太优雅）可以添加一个新的匹配子句到路由中：

```ruby
match 'user_root' => 'users#show'
```
