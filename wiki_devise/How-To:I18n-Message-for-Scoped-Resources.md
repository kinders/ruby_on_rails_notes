# 作用域资源的国际化信息
I have two scoped users: `users` and `students`.
我有两个作用用户：`users`和`students`。

I each have a different authentication key for logging in: `email` and `user_id`.
用不同的认证键来登录：`email`和`user_id`。

I want the different messages to popup when they try to login and fail.
登录或失败时弹出不同信息：

  ```ruby
   devise:
    failure:
      user:
        invalid: 'Invalid email or password.'
      student:
        invalid: 'Invalid user ID or password.'
   ```
