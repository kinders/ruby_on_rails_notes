# 将devise设置为单用户系统

Rationale: Some projects might come across the need for an authentication solution to which devise is supremely suited for -- but without the need (or want) to have public viewers trying to register.
基本原理：一些项目可能只是需要一个认证解决方案，devise特别合适——但不需要公共观众来注册。
 
The example of a private weblog comes to mind, so we will use it as an example.
比如私人博客，下面就用它来做例子。

This, along with [[How to: add an admin role]] (Especially using Option 2 of just using an `:admin` attribute), and lastly **How To: Restrict access to specific actions** gives a pretty robust authentication and authorization abilities to your app/website in mere minutes.
这连同《添加一个管理员角色》（特别是使用选项二“使用一个`:admin`属性”中），最新的《限制访问特定方法》，仅仅几分钟就给了一个相当健壮的认证和授权能力到程序/网站。

 
In general, you want the drop-in functionality Devise give us, sans "allowing the general public to register".
一般，你需要的Devise给的拜访功能，没有“允许一般公共用户来注册”。
You could **Require admin to activate account before sign in**, but that's only dealing with the issue after they have registered.
可以要求管理员在登录之前激活账户，但只是在他们注册之后处理问题的。
Same issue comes with just hiding the links to the pages you are trying to block.
同样的问题会出现，只是隐藏了通向想要阻止的页面的链接。
 
Depending on how you deploy out your application, you may wish to create the sole user before you make the needed changes.
取决于你怎么部署程序，你可能希望在必要的改变之前创建核心用户。
I'll leave that to the reader how they want to do it, but most likely it will be via `rails console` or via a migration file, if not done in development before you remove the functionality.
我会留话告诉读者怎么做，但最常见的恐怕是通过rails控制台或通过一个迁移文件，如果开发时在删除该功能之前还没有完成。
 
## Step 1

Alter the `devise_for` line in config/routes.rb to skip the registration controller:
在路由文件中改变`devise_for`行，跳过注册控制器：

```ruby
devise_for :users, :skip => :registrations
```

## Step 2

Since the base Devise controllers (and views) assume you have registrations enabled, you'll have to override the default views to not show links to the `:sign_up` route (in particular).
因为基本的Devise控制器（和视图）假设你已经开启了注册，所以你必须改写默认的视图，不要显示链到注册路由的链接。
Otherwise, the log in page will throw errors about the routes we skipped in step 1 not existing.
否则，日志将会抛出错误：我们刚跳过的第一步的路由并不存在。
 
In a terminal:
在终端：

```
mkdir -p app/views/devise/shared
touch app/views/devise/shared/_links.html.erb
```

You may wish to keep the link to the `forgot_password` functionality (in case you do forget your password), in which case just add the following to app/views/devise/shared/_links.html.erb:
你可能想保持“忘记密码”的链接（确实有可能忘记密码），这样只需添加下面的代码到app/views/devise/shared/_links.html.erb中：

```erb
<%- if devise_mapping.recoverable? && controller_name != 'passwords' %>
  <%= link_to "Forgot your password?", new_password_path(resource_name) %>
<% end -%>
```

## That's it

The `/users/sign_in` path still gives you a login form, and `/users/sign_out` still logs you out.
`/users/sign_in`路径仍然让一个登录的表单，`/users/sign_out`仍会让你登出。

You can add this to your application.html.erb to make some quick and easy links:
可以将下面添加到application.html.erb来做一些快速简单的链接：

```erb
<% if user_signed_in? %>
   <%= link_to('logout', destroy_user_session_path) %>
<% else %>
  <%= link_to('login', new_user_session_path) %>
<% end %>
```
