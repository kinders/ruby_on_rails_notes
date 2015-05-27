#  通过轻目录访问协议认证
In `config/initializers/` add a new file `ldap_authenticatable.rb`.
在 `config/initializers/` 添加一个新文件 `ldap_authenticatable.rb`。

```ruby
require 'net/ldap'
require 'devise/strategies/authenticatable'

module Devise
  module Strategies
    class LdapAuthenticatable < Authenticatable
      def authenticate!
        if params[:user]
          ldap = Net::LDAP.new
          ldap.host = [YOUR LDAP HOSTNAME]
          ldap.port = [YOUR LDAP HOSTNAME PORT]
          ldap.auth email, password
        
          if ldap.bind
            user = User.find_or_create_by(email: email)
            success!(user)
          else
            fail(:invalid_login)
          end
        end
      end
      
      def email
        params[:user][:email]
      end

      def password
        params[:user][:password]
      end

    end
  end
end

Warden::Strategies.add(:ldap_authenticatable, Devise::Strategies::LdapAuthenticatable)
```

Add a configuration option in `config/initializers/devise.rb`
在`config/initializers/devise.rb`添加一个配置选项。

```ruby
Devise.setup do |config|
  config.warden do |manager|
    manager.default_strategies(:scope => :user).unshift :ldap_authenticatable
  end
# ... rest of devise config
end
```
