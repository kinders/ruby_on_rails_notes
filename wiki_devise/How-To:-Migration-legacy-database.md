migrate a legacy user database to devise, which is using password MD5 hexdigest without salt, put this code in initializers:

```ruby
module Devise
  module Models
    DatabaseAuthenticatable.class_eval do      
      def valid_password_with_legacy?(password)
        if self.legacy_password_hash.present?
          if ::Digest::MD5.hexdigest(password).upcase == self.legacy_password_hash
            self.password = password
            self.legacy_password_hash = nil
            self.save!
            return true
          else
            return false
          end
        else
          return valid_password_without_legacy?(password)
        end
      end

      alias_method_chain :valid_password?, :legacy
    end

    Recoverable.class_eval do
      def reset_password_with_legacy!(new_password, new_password_confirmation)
        self.legacy_password_hash = nil
        reset_password_without_legacy!(new_password, new_password_confirmation)
      end

      alias_method_chain :reset_password!, :legacy
    end
  end
end
```

***

Josevalim suggested a much better solution in ([issue #1858](https://github.com/plataformatec/devise/issues/1858)), just override valid_password? and reset_password? and use 'super' in your devise model

```ruby
    class User
      def valid_password?(password)
        if self.legacy_password_hash.present?
          if ::Digest::MD5.hexdigest(password).upcase == self.legacy_password_hash
            self.password = password
            self.legacy_password_hash = nil
            self.save!
            true
          else
            false
          end
        else
          super
        end
      end

      def reset_password!(*args)
        self.legacy_password_hash = nil
        super
      end
    end
```

## Migrating away from plain text passwords

If you are migrating from a system containing unencrypted plain text passwords, it is straight forward to update all of the records without waiting for some users to log in. I did this in the rails console:

    users = User.where("legacy_password"!=''); users.each do |i|; i.valid_password?(i.legacy_password); end

This makes two assumptions:

1. That you called your legacy password field "legacy_password" instead of "legacy_password_hash" and changed all references in the methods.

2. Instead of doing `::Digest::MD5.hexdigest(password).upcase == self.legacy_password_hash` you can simply replace it with `password == self.legacy_password`

3. That your legacy data will satisfy your validation criteria on save. I had an issue of some legacy passwords being only 4 characters long, so you would have to reduce the min-length in devise.rb to allow successful saving. Some of my legacy fields were also blank when in the new system they were required, so you may need to set defaults as part of the console script.

You can then delete your legacy_password column, as all the passwords will now be encrypted. Before doing this check that you don't have any non-nil legacy_password instances which need manual correction before running the console script again.