# vim: set ft=markdown :#
As of Devise 1.2.rc (see [[https://github.com/plataformatec/devise/pull/670]]), you can make your devise users' emails case-insensitive by changing config/initializers/devise.rb to use:
到了Devise 1.2.rc，可以让devise用户邮件地址对大小写不敏感，使用config/initializers/devise.rb：

```ruby
...
config.case_insensitive_keys = [:email]
...
```

This works for forgot password, sign up, sign in, etc.
这在忘记密码、注册和登录等等都可用。

Replace `:email` with the authentication key that you use.
用你所使用的认证键来替换`:email`。

If you need to find a user (for example in a custom sessions_controller), note that `User.find_by_email(params[:email])` is case sensitive.
Use `User.find_for_authentication(:email =>  params[:email])` instead.
如果需要找到一个用户（比如在一个自定义的会话控制器中），注意`User.find_by_email(params[:email])`是大小写敏感的。
可以使用`User.find_for_authentication(:email =>  params[:email])`。

