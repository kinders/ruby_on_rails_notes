When working with a legacy database where a non-supported password encryption method is used, you can write your own custom encryptor.

## Devise version 2.1 and up

You have to add the [devise-encryptable](https://github.com/plataformatec/devise-encryptable) in addition to the devise gem.

```ruby
# Gemfile
gem 'devise-encryptable'
```

Implement your own encryptor.

```ruby
# config/initializers/md5.rb
require 'digest/md5'

module Devise
  module Encryptable
    module Encryptors
      class Md5 < Base
        def self.digest(password, stretches, salt, pepper)
          str = [password, salt].flatten.compact.join
          Digest::MD5.hexdigest(str)
        end
      end
    end
  end
end
```

## Devise versions prior to 2.1

Implement your own encryptor.

```ruby
# lib/devise/encryptors/md5.rb
require 'digest/md5'

module Devise
  module Encryptors
    class Md5 < Base
      def self.digest(password, stretches, salt, pepper)
        str = [password, salt].flatten.compact.join
        Digest::MD5.hexdigest(str)
      end
    end
  end
end
```

You can then set this as your encryptor in `config/initializers/devise.rb`:

```ruby
config.encryptor = :md5
```

Don’t forget to enable the `:encryptable` in your User model.

Also, you should make sure that the new file is loaded, for instance by adding this to your users' class:

```ruby
require Rails.root.join('lib', 'devise', 'encryptors', 'md5')
```

See also: http://www.markrichman.com/2010/11/22/rails-devise-datamapper-authentication/

## Problems?

Additionally, I had to add the following to my user model in order to be able to log in. Supposedly there is a more elegant way, but I did not get `devise_for … :encryptor => :md5` to work, nor `devise :encryptor => :md5`. Maybe this is because I do not have a password salt.

Try adding :encryptable to your devise statement as well. `devise :encryptable, :encryptor => :md5`.

```ruby
# app/models/user.rb 
# debt: we should not need to do this, but seems like setting :encryptor => :md5 on the devise or devise_for
# does not do anything. Maybe because we don't use a salt. So this works for now.
def valid_password?(password)
  return false if encrypted_password.blank?
  Devise.secure_compare(Devise::Encryptors::Md5.digest(password, nil, nil, nil), self.encrypted_password)
end
```

And how I tested it.

```ruby
# spec/models/user_spec.rb
describe "devise valid_password?" do
  it "should use our hashing mechanism, not the default bcrypt" do
    Factory(:member).valid_password?('blahblah').should be_true
  end
end
```

## Workaround for empty salt

Encryptable expects a `password_salt` attribute in the model, else it won't even call the custom encryptor or even cause exceptions:

```ruby
# app/models/user.rb
def password_salt
  'no salt'
end

def password_salt=(new_salt)
end


```
**This advice of 'working around' the absence of a salt is not sound.  Not only is MD5 no longer safe http://www.kb.cert.org/vuls/id/836068 (SHA2 > 256 key length is recommended see http://en.wikipedia.org/wiki/SHA-2) but even then hashes without a salt are open to rainbow table (brute force) attacks.  Don't become another LinkedIn, use a strong hashing function with a salt.**