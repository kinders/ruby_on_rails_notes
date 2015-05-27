# 允许用户不提供密码编辑账户信息
By default, Devise allows users to change their password using the `:registerable` module.
默认情况测，Devise使用`:registerable`模块允许账户改变密码。
But sometimes, developers want to create other actions that allow the user to change their information without requiring a password.
但有时，开发者想创建其他动作允许用户无需提供密码来改变用户信息。
The best option in this case is to create your own controller, that belongs to your application, and provide an edit and update actions, as you would do for any other resource in your application.
这种情况下最好的选项是创建自己的控制器，提供一个编辑和更新动作，和程序的其他资源模型一样。
 
Keep in mind though to be restrictive in the parameters you allow to be changed.
记住要严格选择允许改变的参数。
In particular, you likely want to permit just user data fields and avoid e-mail, password and such information to be changed: 
特别是，你可能想允许修改用户数据字段和有效电邮、密码诸如此类的信息。

    params[:user].permit(:first_name, :last_name, :address)

The other solution would be to simply override "update resource" method in your registrations controller like that
另一种方案是改写注册控制器的“更新资源”方法。

```
class RegistrationsController < Devise::RegistrationsController

  protected

  def update_resource(resource, params)
    resource.update_without_password(params)
  end
end

```

And don't forget to tell devise use your controller in routes.rb: 
不要忘了告诉devise使用路由的控制器：

    devise_for :users, controllers: {registrations: 'registrations'}

