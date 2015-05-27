# 用Devise做一个自定义邮件验证器

Note: This approach has been used with Devise 1.5.4 under Rails 3.2.12, YMMV.
注意：这个方法已经被用于Rails 3.2.12的Devise 1.5.4。

Email validation in a production apps is challenging because the RFC is complex, and in the real world different email sub-systems differ in their adherence to the RFC spec.
It is not uncommon for a "valid" email to be rejected by a system that may validate email addresses with a too-narrow subset of rules.
电邮验证对生产应用是个挑战，因为RFC太复杂，真实世界里邮件子系统不会完全跟随RFC规范。
让有效电邮被系统拒绝是不正常的，可能是对电邮地址设置了过于狭窄的验证规则。

The Devise email validator is intentionally relaxed to reduce the likelihood of rejecting a valid email, but, as a result, may permit users to register with an email address that is not deliverable.
One specific example is `someone@example.co,`.
Whether or not that address is valid according to the RFC, it clearly cannot be delivered.
Devise电邮验证器有意放松来减少拒绝有效电邮的可能，但副作用是可能允许用户用一个不能发送的电邮地址来注册。
比如`someone@example.co,`。
不管该地址根据RFC是否有效，它明显不能发送。
(And since the comma key is next to the "m" key on most keyboards, it's not that hard for a user to type ".co," instead of ".com")
（因为逗号在多数键盘上都临近m键，用户很容易把".com"输入为“.co,”）

Rails permits custom validators, so it is quite simple to add your own custom email validator, and no change to Devise is necessary (as long as you want Devise's built-in validator to also be applied).
Rails 允许自定义验证器，很容易添加你的自定义电邮验证器，并让Devise保持不变是必要的（只要你想Devise的内建验证器也应用起来）
You'll get the union of the two validators, eg, an email has to pass both validators.
比如，可以将两个验证器联合起来，一个电邮通过两个验证器。
If your custom validator is "more strict" than Devise's validator (as in this example), your app will have the benefit of the stricter validation automatically.
如果自定义验证器比Devise的更严格（如下面这个例子），程序会自动得益于这个更严格的验证器。

One approach to a non-regex email validation is documented at 
非正则表达式的电邮验证器的一种方式记载在：

    http://my.rails-royce.org/2010/07/21/email-validation-in-ruby-on-rails-without-regexp/

N.B. If your goal is to replace, not supplement, Devise's email validator, the same blog also outlines how to here:
注意：如果你的目标是取代而不是补充Devise的电邮验证器，这里有个博文可以参考：

    http://my.rails-royce.org/2010/10/20/rails-how-to-skip-or-remove-validations-in-a-model/

If your goal is to supplement, but not replace, the Devise email validation, the approach is simpler:
如果目标是补充，而不是取代Devise的电邮验证器，方法就更简单了：

1) Add to your User model:
添加到User模型：

```ruby
validates :email, :presence => true, :email => true
```

2) Add to the app/validators directory, copied verbatim from http://my.rails-royce.org/2010/07/21/email-validation-in-ruby-on-rails-without-regexp/
添加到app/validators目录，从～逐字复制过来的：

```ruby
# app/validators/email_validator.rb
require 'mail'
class EmailValidator < ActiveModel::EachValidator
  def validate_each(record,attribute,value)
    begin
      m = Mail::Address.new(value)
      # We must check that value contains a domain and that value is an email address
      # 必须检查该值包含一个域名、是个电邮地址
      r = m.domain && m.address == value
      t = m.__send__(:tree)
      # We need to dig into treetop
      # 我们需要挖到树顶。
      # A valid domain must have dot_atom_text elements size > 1
      # 有效域名必须包含点号文本元素，长度大于1.
      # user@localhost is excluded
      # treetop must respond to domain
      # We exclude valid email values like <user@localhost.com>
      # Hence we use m.__send__(tree).domain
      r &&= (t.domain.dot_atom_text.elements.size > 1)
    rescue Exception => e   
      r = false
    end
    record.errors[attribute] << (options[:message] || "is invalid") unless r
  end
end
```

3. (Be sure to restart your server)
重启服务器


