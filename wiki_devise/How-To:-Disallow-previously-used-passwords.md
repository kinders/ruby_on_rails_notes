The [`devise_security_extensions`](https://github.com/phatworx/devise_security_extension) gem handles this, but the following tutorial will show you how to hand-roll your own implementation:

This is achieved by storing records of previous password hashes, then checking them as a validation. I am using bcrypt on version 1.3.4 here, and rails 3. If you are using a different encryptor or version, then you might have to adjust the methods to suit your needs, but the concept is the same.

Create a new db model to hold the hashes

```rb
# db/migrations/<TIMESTAMP>_create_password_histories.rb
class CreatePasswordHistories < ActiveRecord::Migration
  def self.up
    create_table(:password_histories) do |t|
      t.integer :user_id
      t.string  :encrypted_password
      t.timestamps
    end
  end

  def self.down
    drop_table :password_histories
  end
end
```

Modify the User model to enable the validation, and to save each password in a PasswordHistory model:

```rb
# models/user.rb
  include ActiveModel::Validations
  
  has_many :password_histories
  after_save :store_digest
  validates :password, :unique_password => true

  private
  
  def store_digest
    if encrypted_password_changed?
      PasswordHistory.create(:user => self, :encrypted_password => encrypted_password)
    end
  end
```

Make a class that contains the validation code:

```rb
# models/unique_password_validator.rb
require 'bcrypt'
class UniquePasswordValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    record.password_histories.each do |ph|
      bcrypt = ::BCrypt::Password.new(ph.encrypted_password)
      hashed_value = ::BCrypt::Engine.hash_secret([value, Devise.pepper].join, bcrypt.salt)
      record.errors[attribute] << "has been used previously." and return if hashed_value == ph.encrypted_password
    end
  end
end
```

Thats it! If you are not using bcrypt, look into the Devise source code to figure out how to recreate the `hashed_value` for your validation.