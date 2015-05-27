Two different ways to do this:
两种方法：

**1. Global config for your app**:  程序的全局配置

In `config/initializers/devise.rb`:

> `config.password_length = 10..128` 

**2. On a per-model basis**:   在每个模型中。

In `user.rb`, for example:

> `devise :database_authenticatable, :validatable, password_length: 10..128`
