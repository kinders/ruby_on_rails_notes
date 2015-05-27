Here is how you have to migrate from restful_authentication to Devise.

You'll need to add two gems in Gemfile:
```ruby
  gem 'devise'
  gem 'devise-encryptable'
```

In your **model file**, add the below lines

```ruby

  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :confirmable, :validatable, 
         :encryptable, :encryptor => :restful_authentication_sha1

```

Modules `:encryptable` and `:encryptor => :restful_authentication_sha1` are important here. Other modules are optional as per your app's business logic.

In **config/initializers/devise.rb**, configure as below

```ruby

  config.stretches = 1

  config.pepper = ""

```

We are setting **empty pepper** and **stretches to 1**. This is how restful authentication works. It does not use stretches or pepper (by default).

( Note: You may find REST_AUTH_SITE_KEY in **config/initializers/site_keys.rb** )

In other case if You did set the REST_AUTH_SITE_KEY , then configure as below

```ruby

  config.stretches = 10

  config.pepper = "<YOUR_REST_AUTH_SITE_KEY>"

```

If you're using `:validatable` you may have to adjust your minimum password length. Devise uses **8**, restful_authentication uses **6**. 

In **config/initializers/devise.rb**

```ruby
config.password_length = 6..128
```


That is it for configuration. Now we have to change the table column names (without deleting them) to give devise what it wants. Here is the migration file I used. Change it according to your app's business logic.

```ruby
class ConvertRaToDevise < ActiveRecord::Migration
  def self.up
    
    #encrypting passwords and authentication related fields
    rename_column :accounts, "crypted_password", "encrypted_password"
    change_column :accounts, "encrypted_password", :string, :limit => 128, :default => "", :null => false
    rename_column :accounts, "salt", "password_salt"
    change_column :accounts, "password_salt", :string, :default => "", :null => false
    
    #confirmation related fields
    rename_column :accounts, "activation_code", "confirmation_token"
    rename_column :accounts, "activated_at", "confirmed_at"
    change_column :accounts, "confirmation_token", :string
    add_column    :accounts, "confirmation_sent_at", :datetime

    #reset password related fields
    rename_column :accounts, "password_reset_code", "reset_password_token"
    
    #rememberme related fields
    add_column :accounts, "remember_created_at", :datetime #additional field required for devise.
  
  end

  def self.down
    
    #rememberme related fields
    remove_column :accounts, "remember_created_at"
    
    #reset password related fields
    rename_column :accounts, "reset_password_token", "password_reset_code"
    
    #confirmation related fields
    rename_column :accounts, "confirmation_token", "activation_code"
    rename_column :accounts, "confirmed_at", "activated_at"
    change_column :accounts, "activation_code", :string
    remove_column :accounts, "confirmation_sent_at"

    #encrypting passwords and authentication related fields
    rename_column :accounts, "encrypted_password", "crypted_password"
    change_column :accounts, "crypted_password", :string, :limit => 40
    rename_column :accounts, "password_salt", "salt" 
    change_column :accounts, "salt", :string, :limit => 40
    
  end
end

```

Migrate the database using the above migration file and you are done with restful_authentication to devise migration. 

**Restart your server** to get the config changes take effect. 