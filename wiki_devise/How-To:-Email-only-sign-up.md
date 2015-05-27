# 只用电邮注册

### 前言
For Rails 4, check https://github.com/plataformatec/devise/wiki/How-To:-Override-confirmations-so-users-can-pick-their-own-passwords-as-part-of-confirmation-activation
对于Rails 4，查看《改写确认，让用户在确认激活部分选择自己的密码》。

Sometimes, you want a [gradual engagement](http://www.lukew.com/ff/entry.asp?1130) feature where a visitor signs up using only their email address.
Or maybe an admin creates user accounts when a client contacts the business.
Whatever the logic, the user creates their password after their account is created.
有时，你想一个《渐进约会》的特征，访问者只使用它们的电邮来注册。
或者客户联系生意时，管理员创建了用户账户。
 
Users will get an email asking them to confirm their email address.
A link in the email takes them to a webpage that asks them to set their password.
If the user does not set their password, they are not confirmed, they must set their password in order to confirm their account.
用户将得到一封信来确认电邮地址。
电邮里的链接跳转到一个网页，让它们设置自己的密码。
如果用户没有设置密码，账户就不能得到确认。
 
You can also keep the default Devise sign up where a new user enters both an email and password.
In that case, just skip the first step.
也可以保持Devise的默认注册，新用户需要输入电邮和密码。
那种情况下，就可以跳过第一步了。

Note: if you use multiple user resource tables (like :clients and :admins) see the example at the bottom.
注意：如果使用了多个用户资源表（比如:clients和admins），可参见底部的示例。

(This technique was first documented by [Claudio Marai](http://blog.devinterface.com/2011/05/two-step-signup-with-devise/).
The following steps combine Claudio's example code with code contributed in the comments section of his post and updates everything for Devise 2.
（这项技术最先为Claudio Marai所提出。）
下面的步骤结合了Claudio 的例子代码和他的作品下的注释里的贡献代码，并为Devise 2作了升级。

**NOTE: This is for Rails 3 and Devise ~> 2.0**.
**注意：这适用于Rails4 和Devise 2.0系列**

The following view code uses [formtastic](https://github.com/justinfrench/formtastic) and [haml](https://github.com/haml/haml) gems because they make the code cleaner, with less typing.
下面的视图代码使用了formtastic和haml软件包，为了让代码更干净，减少输入量。
See [simple_form](https://github.com/plataformatec/simple_form) for another easy way to create forms.)
另一种简单的方法参见simple_form。

### 1. 询问电邮
Modify the #new view in app/view/users/registrations (optional step)
在app/view/users/registrations修改#new视图（可选步骤）

This step is optional: This modification allows a user to sign up with only their email address.
这一步是可选的：这个修改允许用户只用电邮地址注册。
Skip this step if you're only interested in allowing admins to create users with only an email address.
如果只是让管理员用电邮来创建用户，可以跳过这一步。

You want to make it easy for new users to sign up -- just ask for their email address and not worry about passwords for now.
需要让新用户感到这一步易于登录——现在只是询问电邮地址，无需密码。
That's the "gradual engagement" approach!
这就是《渐进约会》的方式！

```haml
    %h2 Sign up.
All we need is your email address.
    = semantic_form_for(resource, :as => resource_name, :url => user_registration_path(resource)) do |form|
      = devise_error_messages!
      = form.inputs do
        = form.input :email, :input_html => {:autofocus => true}
      = form.actions do
        = form.action :submit, :label => "Sign up"
    = render 'shared/links'
```
### 2. 询问密码
Create a #show view in app/view/confirmations
在app/view/confirmations创建一个#show视图。

In this view, we ask the new user to create a password and confirm it.
在这个视图，我们请新用户创建一个密码并确认它。
We embed the confirmation_token in a hidden input field so that the controller will receive it.
我们在一个隐藏的输入框中嵌入了confirmation_token，以便控制器重新获得它。
(We'll explain the confirm_path in a later step):
（稍后的步骤将解释confirm_path）

```haml
    %h2 You're almost done! Now create a password to securely access your account.
    = semantic_form_for(resource, :as => resource_name, :url => confirm_path) do |form|
      = devise_error_messages!
      = form.inputs do
        = form.input :password, :input_html => {:autofocus => true}
        = form.input :password_confirmation
        = form.input :confirmation_token, :as => :hidden
      = form.actions do
        = form.action :submit, :label => 'Confirm Account'
```

**For Rails 4/Devise 3**:
**使用Rails 4/Devise 3**:

```haml
    %h2 You're almost done! Now create a password to securely access your account.
    = semantic_form_for(resource, :as => resource_name, :url => confirm_path) do |form|
      = devise_error_messages!
      = form.inputs do
        = form.input :password, :input_html => {:autofocus => true}
        = form.input :password_confirmation
        = form.input :confirmation_token, :as => :hidden, :input_html => { :value => @original_token }
      = form.actions do
        = form.action :submit, :label => 'Confirm Account'
```


### 3. 密码检验
Create a #password_match? method and overwrite Devise's #password_required?
创建一个 #password_match? 方法并改写Devise的#password_required?方法。

We need a convenient way to test that :password_confirmation matches :password and report any errors.
需要一个简便的方法来测试那个:password_confirmation是否匹配:password，不匹配则报告错误。
We also need to overwrite Devise's #password_required? method so the user can sign up without specifying a password.
也需要改写Devise的#password_required?方法，以便用户可以不用密码注册。
So in your model, add these public methods:
所以，在模型中添加这些公共方法：

```ruby
    class User < ActiveRecord::Base
      def password_required?
        super if confirmed?
      end

      def password_match?
        self.errors[:password] << "can't be blank" if password.blank?
        self.errors[:password_confirmation] << "can't be blank" if password_confirmation.blank?
        self.errors[:password_confirmation] << "does not match password" if password != password_confirmation
        password == password_confirmation && !password.blank?
      end
    end
```

### 4. 处理确认
Overwrite the Devise confirmations controller's #show
改写Devise确认控制器的#show

Create a new controller app/controllers/confirmations_controller.rb to overwrite Devise's #show.
创建一个新的控制器app/controllers/confirmations_controller.rb来改写Devise的#show。
If the user is already confirmed, they are only confirming a change in email address so call super.
如果用户已经确认，他们只确认电邮地址的更改，以便调用super。
And do you remember calling #confirm_path in the #show view? 
还记得在#show视图中调用#confirm_path吗？
The #confirm method will be called when the user submits the form.
用户提交表单时，#confirm方法将被调用。
But we'll only confirm the user if :password matches :password_confirmation, otherwise re-render the #show view so the user can try again.
但只有:password_confirmation和:password匹配时才能确认，否则会重新呈现#show视图，让用户再试一次。
Calling `super` if the user is already confirmed makes the first step (above) optional and only needed if you want a ["gradual engagement"](http://www.lukew.com/ff/entry.asp?1130) sign-up.
如果用户已经得到确认，调用`super`，对于上面的第一步是可选的，只是想要一个“渐进约会”式的注册时才需要。

```ruby
    class ConfirmationsController < Devise::ConfirmationsController
      def show
        self.resource = resource_class.find_by_confirmation_token(params[:confirmation_token]) if params[:confirmation_token].present?
        super if resource.nil? or resource.confirmed?
      end

      def confirm
        self.resource = resource_class.find_by_confirmation_token(params[resource_name][:confirmation_token]) if params[resource_name][:confirmation_token].present?
        if resource.update_attributes(params[resource_name].except(:confirmation_token)) && resource.password_match?
          self.resource = resource_class.confirm_by_token(params[resource_name][:confirmation_token])
          set_flash_message :notice, :confirmed
          sign_in_and_redirect(resource_name, resource)
        else
          render :action => "show"
        end
      end
    end
```

**For Rails 4/Devise 3**: Strong attributes will require `permit`, so be sure to include the passed parameters on the `update_attributes`.
**对于 Rails 4/Devise 3**: 健壮属性将要求`permit`，所有确保在`update_attributes`上包含传递的参数。

```ruby
    class ConfirmationsController < Devise::ConfirmationsController

      def show
        if params[:confirmation_token].present?
          @original_token = params[:confirmation_token]
        elsif params[resource_name].try(:[], :confirmation_token).present?
          @original_token = params[resource_name][:confirmation_token]
        end

        self.resource = resource_class.find_by_confirmation_token Devise.token_generator.
          digest(self, :confirmation_token, @original_token)

        super if resource.nil? or resource.confirmed?
      end

      def confirm
        @original_token = params[resource_name].try(:[], :confirmation_token)
        digested_token = Devise.token_generator.digest(self, :confirmation_token, @original_token)
        self.resource = resource_class.find_by_confirmation_token! digested_token
        resource.assign_attributes(permitted_params) unless params[resource_name].nil?

        if resource.valid? && resource.password_match?
          self.resource.confirm!
          set_flash_message :notice, :confirmed
          sign_in_and_redirect resource_name, resource
        else
          render :action => 'show'
        end
      end

     private
       def permitted_params
         params.require(resource_name).permit(:confirmation_token, :password, :password_confirmation)
       end
    end
```

### 5. 告诉Devise
Tell Devise to use the new controller
告诉Devise使用新的控制器：

Finally, tell Devise about the new controller and its #confirm action.
最后，告诉Devise关于新的控制器和确认动作。
(Notice the pluralized resource name in the #devise_for method call and the singular resource name in the #devise_scope method call):
（注意在devise_for方法调用里的复数资源名称、#devise_scope方法调用里的单数资源名称）

```ruby
    devise_for :users, :controllers => {:confirmations => 'confirmations'}

    devise_scope :user do
      put "/confirm" => "confirmations#confirm"
    end
```

**For Rails 4 with Devise 3** use `patch "/confirm" => "confirmations#confirm"`.
**对于Rails 4 with Devise 3** 使用`patch "/confirm" => "confirmations#confirm"`。

## Examples for multiple resources 多资源示例

Here's an example for multiple resources, which is why we use `resource` and `resource_name` and `resource_class` in the confirmations controller.
这是一个多资源的示例，这也是为什么在确认控制器中使用`resource`、`resource_name`和`resource_class`的原因。
We use named routes so Devise can detect the different resource scopes and use the same confirmations controller.
我们使用命名路由，所有Devise可以检测到不同的资源作用域，并使用同样的确认控制器。

First, the routes:
首先，路由：

```ruby
    devise_for :clients,     :controllers => {:confirmations => 'confirmations'}
    devise_for :admins,      :controllers => {:confirmations => 'confirmations'}
    devise_for :supervisors, :controllers => {:confirmations => 'confirmations'}

    devise_scope :client do
      put "/clients/confirm" => "confirmations#confirm", :as => :client_confirm
    end
    devise_scope :admin do
      put "/admins/confirm" => "confirmations#confirm", :as => :admin_confirm
    end
    devise_scope :supervisor do
      put "/supervisors/confirm" => "confirmations#confirm", :as => :supervisor_confirm
    end
```

You have to create app/views/clients/registrations/new.html.haml, app/views/admins/registrations/new.html.haml, and app/views/supervisor/registrations/new.html.haml templates.
必须创建app/views/clients/registrations/new.html.haml, app/views/admins/registrations/new.html.haml,和 app/views/supervisor/registrations/new.html.haml模板。

Next, the #show view (we just change the confirm_path helper call):
下一步，#show视图（只是改变confirm_path辅助器调用）

```haml
    %h2 You're almost done! Now create a password to securely access your account.
    = semantic_form_for(resource, :as => resource_name, :url => send(:"#{resource_name}_confirm_path")) do |form|   # 就是这里的send的参数
      = devise_error_messages!
      = form.inputs do
        = form.input :password, :input_html => {:autofocus => true}
        = form.input :password_confirmation
        = form.input :confirmation_token, :as => :hidden
      = form.actions do
        = form.action :submit, :label => 'Confirm Account'
```
