It's now increasingly common for websites to provide "super fast & easy" registration : the user just gives his e-mail.
现在越来越少网站提供只需要电子邮箱的“超级快速简便”的注册了。
The confirmation e-mail then contains a password generated for the user.
注册之后的确认电邮会包含一个用户密码。
If the user is not happy with the password (either wants a super-weak or super-strong one), the ability to change it is given right away when accessing the confirmation link.
如果用户不喜欢这个密码（太弱或者太强），访问确认连接时，下面会给出改变的能力。

This method is used for example by slashdot.org:
这种方法用于，比如slashdot.org：

```ruby
generated_password = Devise.friendly_token.first(8)
user = User.create!(:email => email, :password => generated_password)

RegistrationMailer.welcome(user, generated_password).deliver
```

QUESTION:
问题：

Are you overriding the Devise Registration Controller Create action here? 
这里是否改写了Devise注册控制器的创建动作？
Could you provide the full controller to show how you have implemented this? 
可以提供完整的控制器，看看它是怎么实现的吗？
Could you also provide the details of your welcome message that would include both the newly created password? 
可以提供欢迎信息的细节，它里面是否包含了新建的密码？
And the details of your confirmation page that includes the request to change their password?
确认页面的细节是否包含了更改密码的请求？

ANSWER:

I'm afraid my answer won't be so simple or much more illuminating.
恐怕答案没这么简单，或没那么有用。
I am not overriding the Devise Registration Controller, I do not use it at all.
没有改写Devise的注册控制器，根本就没有到它。
There is no way for a user to directly create an account in my system.
没有办法让用户直接在系统上创建一个账户。
They go through a workflow, and at the end of the workflow, I create their account using the code above and send them an email.
它们通过了一个工作流，在该流的末端，我使用上面的代码创建了一个账户，然后发给它们一封信。
I follow the above code with
我用下面的代码跟随上面的代码。

``` ruby
sign_in(:user, user)
```

so they are just logged in as a new user seamlessly.
所以他们只是作为新用户登录。
The log in is just for when they come back.
登录只是在他们回来时才会用到。
In the email, I include a link to /user/edit to change their password, but I don't require it, nor do I require confirmation by email.
在电邮中，我包含了一个到/user/edit的链接来更改他们的代码，但我不需要它，也不需要通过电邮确认。
I'll write something up in a blog post this weekend so you can see a working example.
我将会在博客中写一些贴文，你就可以看到一个可用的示例了。

Note:  I don't believe our setup here is the most secure way to go, but we're not accepting payment information from our website, and we have a sales/support team that speaks with everyone who uses our system.
注意： 我不相信我们这里的设置是最安全的，但不能接受来自从站点的付款信息，但我们有一个销售/支持小组和使用这个系统的用户沟通。
There's limited scope for an impostor/hacker to do much damage with their login.
这里限制了骗子和黑客利用登录搞搞破坏的范围。
 No, that is not a dare or challenge.
不，那不是挑战。
