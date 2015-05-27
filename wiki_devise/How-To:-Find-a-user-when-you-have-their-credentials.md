# vim: set ft=markdown :
# 通过信任状找到用户
From https://github.com/plataformatec/devise/issues/346, you can do the following:
来自问题346，可以这样写：

    user = User.find_for_authentication(:username => 'druidia')
    user.valid_password?('12345')  # That's amazing! I have the same combination on my luggage!

Here's a convenient method for same:
下面是一个更方便的方法：

    class User
      def self.authenticate(username, password)
        user = User.find_for_authentication(:username => username)
        user.valid_password?(password) ? user : nil
      end
    end

Addendum: If you use confirmable, and don't want to authenticate the user if she isn't confirmed yet, you also need to call "active_for_authentication?"
附录：如果使用了确认模块，并且因为用户还未确认而不想认证用户，需要调用`active_for_authentication?`
