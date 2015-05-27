**Note**: this setting is now provided by default!
注意：这些设置现在已经是默认的了！

---

In your configuration (devise.rb initializer), set the number of stretches to 1 for your test environment:

    config.stretches = Rails.env.test? ? 1 : 10

This will increase performance dramatically if you use bcrypt and create a lot of users (i.e. you use FactoryGirl or Machinist).
