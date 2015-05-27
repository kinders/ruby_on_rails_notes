# 使用自定义邮箱
To use a custom mailer, create a class that extends `Devise::Mailer`, like this:
要使用自定义邮箱，创建一类扩展`Devise::Mailer`，类似下面：

```ruby
class MyMailer < Devise::Mailer   
  helper :application # gives access to all helpers defined within `application_helper`.
  include Devise::Controllers::UrlHelpers # Optional. eg. `confirmation_url`
  default template_path: 'devise/mailer' # to make sure that your mailer uses the devise views
end
```

Then, in your `config/initializers/devise.rb`, set `config.mailer` to `"MyMailer"`.
然后在`config/initializers/devise.rb`，将`config.mailer`设置为`"MyMailer"`。

You may now use your `MyMailer` in the same way as any other mailer.
可以和其他邮箱那样使用`MyMailer`。
In case you want to override specific mails to add extra headers, you can do so by simply overriding the method and calling `super` after (triggering Devise's default behavior).
如果想改写指定的邮件，添加额外的头信息，可以通过改写方法来实现，之后调用`super`（触发Devise的默认行为）。
For instance, we can add a new header for the confirmation_instructions e-mail as follow:
比如，可以像下面那样为确认指令电邮添加一个新的信头：

```ruby
def confirmation_instructions(record, token, opts={})
  headers["Custom-header"] = "Bar"
  super
end
```

You can also override any of the basic headers (from, reply_to, etc) by manually setting the options hash:
也可以通过手动设置选项散列来改写基本信头（从，回复，等等）：

```ruby
def confirmation_instructions(record, token, opts={})
  headers["Custom-header"] = "Bar"
  opts[:from] = 'my_custom_from@domain.com'
  opts[:reply_to] = 'my_custom_from@domain.com'
  super
end
```
