# 允许用户编辑密码
By default, Devise allows users to change their password using the registerable module. 
默认，Devise允许用户使用registerable模块改变密码。

Here we are going to provide a few solutions on how to allow users to change their password.
这里提供一些解决方案，怎么允许用户改变密码。

**Solution 1.**

Just make such link in your view:
只需要在视图添加这样的链接：

```erb
<%= link_to "Change your password", edit_user_registration_path %>
```

Notice: This'll work if you didn't do any modification in your **routes.rb** file such as 
注意：如果路由文件中的语句没有被修改，这个方案是可行的。

`devise_for :users, :skip => [:registrations]`

Notice 2: If you use the latest Devise with Strong Parameters, you should add this line to your ApplicationController.rb :
第二个注意：如果使用最新的使用强壮参数的Devise，应该将这一行增加到程序控制器中：

```ruby
class ApplicationController < ActionController::Base
  before_filter :configure_permitted_parameters, if: :devise_controller?

  def configure_permitted_parameters
    devise_parameter_sanitizer.for(:account_update) { |u| 
      u.permit(:password, :password_confirmation, :current_password) 
    }
  end
end

```
 
**Solution 2.**

Lets suppose that you don't want to allow to sign up but you want to allow to change password for registered users. 
Just paste this code in **routes.rb**:
假如你不允许注册，但允许已注册用户修改密码，
将这些代码复制到路由中：

```ruby
devise_for :users, :skip => [:registrations]                                          
    as :user do
      get 'users/edit' => 'devise/registrations#edit', :as => 'edit_user_registration'    
      put 'users/:id' => 'devise/registrations#update', :as => 'user_registration'            
    end
```

And then you can make such link in your view:
然后可以在视图中加入这个链接：

```ruby
= link_to "Change your password", edit_user_registration_path
```

Notice: you will need to update default devise views accordingly, i.e. in `app/views/devise/registrations/edit.html.erb` 
change `registration_path(resource_name)` to `user_registration_path(resource)`
 (If using shared views for multiple models, you can use `send("#{resource_name}_registration_path", resource)`)`
注意：你需要更新相应的默认devise视图，比如在app/views/devise/registrations/edit.html.erb里，
将`registration_path(resource_name)`改为user_registration_path(resource)，
（如果在多个模型中使用了共享视图，可以使用`send("#{resource_name}_registration_path", resource)`）

_Notice: If you are using rails 4.0+ you should be using patch instead of put for updates. 
You should change the method in the form_tag residing in `app/views/devise/registrations/edit.html.erb` and the `routes.rb` file._
注意：如果使用Rails 4.0系列，应使用patch而不是put来更新。
更新`app/views/devise/registrations/edit.html.erb`和路由里的form_tag方法。

**Solution 3.**

But sometimes, developers want to provide their custom actions that change the password. In such cases, the best option is for you to create manually a controller:
但有时，开发者想提供更改密码的默认动作。
这种情况下，最好的选项是手动创建一个控制器：

```ruby
class UsersController < ApplicationController
  
  before_filter :authenticate_user!

  def edit
    @user = current_user
  end

  def update_password
    @user = User.find(current_user.id)
    if @user.update(user_params)
      # Sign in the user by passing validation in case their password changed
      # 如果更改了密码，通过传入验证登录用户。
      sign_in @user, :bypass => true
      redirect_to root_path
    else
      render "edit"
    end
  end

  private

  def user_params
    # NOTE: Using `strong_parameters` gem
    params.required(:user).permit(:password, :password_confirmation)
  end
end
```

The route should be the following:
路由该是下面的：

```ruby
resource :user, only: [:edit] do
  collection do
    patch 'update_password'
  end
end
```

And then proceed to implement the view, as below:
然后完成视图：像下面：

```erb
<%= form_for(@user, :url => { :action => "update_password" } ) do |f| %>    # 主要是这一行
  <div class="field">
    <%= f.label :password, "Password" %><br />
    <%= f.password_field :password, :autocomplete => "off"  %>
  </div>
  <div class="field">
    <%= f.label :password_confirmation %><br />
    <%= f.password_field :password_confirmation %>
  </div>
  <div class="action_container">
    <%= f.submit %>
  </div>
<% end %>
```

To use "`confirm_password`" field to force user to enter old password before updating with the new one: Change `@user.update(user_params)` to `@user.update_with_password(user_params)` in the controller along with adding `:current_password` to the permitted parameters, then and add the following to the view code:
在新密码更新之前，要使用`confirm_password`字段来强制用户输入旧密码：在控制器中将`@user.update(user_params)`改为`@user.update_with_password(user_params)`，并添加到许可参数中，然后将下面视图代码添加到视图中：

```erb
  <div class="field">
    <%= f.label :current_password %> <i>(we need your current password to confirm your changes)</i><br />
    <%= f.password_field :current_password %>
  </div>
```

Remember, Devise models are like any model in your application.
If you want to provide custom behavior, just implement new actions and new controllers.
Don't try to bend Devise.
记住，Devise模型和程序的其他模型一样。
如果想提供自定义行为，只需实现新动作和控制器。
不过可别想降伏Devise哦。
