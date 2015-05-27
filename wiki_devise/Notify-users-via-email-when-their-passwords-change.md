For security purposes, sometimes you need to notify users when their passwords change.
为了安全起见，有时需要在用户密码更改时提醒用户。
The following code has been tested with Rails 4.0.4 and Devise 3.2.4, assuming your Devise model is named User.
下面的代码已经在Rails 4.0.4和Devise 3.2.4上得到测试，假设Devise模型命名为User。

To do so, you need to generate a new mailer. Let's call it UserMailer:
需要生成一个新的电邮器，就叫UserMailer：

```bash
rails g mailer user_mailer password_changed
```

Add some code:
添加一些代码：

```ruby
# app/mailers/user_mailer.rb
class UserMailer < ActionMailer::Base
  default from: "some-email@your-domain.ext"

  def password_changed id
    @user = User.find(id)
    mail to: @user.email, subject: "Your password has changed"
  end
end
```

Then add some content to the email template:
单后添加内容到电邮模板中：

```ruby
<% #app/views/user_mailer.html.erb %>
<h2>Your password has changed</h2>
<hr>
<p>Hi <%= @user.email %>,</p>
<p>We wanted to let you know that your password was changed.</p>
```

Now configure your model:
现在配置模型

```ruby
# app/models/user.rb
class User < ActiveRecord::Base
  # Overridden to notify users with password changes
  # 改写，密码更改时通知用户
  def update_with_password(params, *options)
    if super
      # TODO schedule this in the background
      UserMailer.password_changed(self.id).deliver
    end
  end
end
```

Voila!
就行了！
