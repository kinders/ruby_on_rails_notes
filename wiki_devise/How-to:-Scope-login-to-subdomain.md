## Overview

1. Modify the Devise-generated migration to remove the index on email uniqueness constraint
2. Change login keys to include `:subdomain`
3. Override Devise hook method `find_for_authentication`

### Modify migration

First, you must remove the email index uniqueness constraint. Because we'll be turning this into a scoped query, we will scope the new index to subdomain. If this is a brand new Devise model, you can open the Devise migration and change the following:
```ruby
# db/migrate/XXX_devise_create_users.rb
def change
  # Remove this line
  add_index :users, :email, :unique => true

  # Replace with
  add_index :users, [:email, :subdomain], :unique => true
end
```
If this is an existing project, you'll need to create a new migration removing the old index and adding a new one:
```sh
rails g migration reindex_users_by_email_and_subdomain
```
```ruby
# db/migrate/XXX_reindex_users_by_email_and_subdomain.rb
def change
  remove_index :users, :email
  add_index :users, [:email, :subdomain], :unique => true
end
```

### Change login keys

In your Devise model, add `:subdomain` to `:request_keys`. By default `:request_keys` is set to `[]`.

```ruby
   # app/models/user.rb
class User
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, request_keys: [:subdomain]
end
```

If you have multiple Devise models and you would like all of them to have the same `:request_keys` configuration, you can set that globally in `config/initializers/devise.rb`
```ruby
config.request_keys = [:subdomain] # default value = []
```

### Check that you do not have :validatable in the devise call on the Model

If you do, :validatable will prevent more than one record having the same email, even in different subdomains.  If you want to keep some of the validations, you can copy the ones you want from https://github.com/plataformatec/devise/blob/master/lib/devise/models/validatable.rb

### Override Devise auth finder hook

For Authenticatable, Devise uses the hook method `Model.find_for_authentication`. Override it to include your additional query parameters:

```ruby
# app/models/user.rb
class User < ActiveRecord::Base
  def self.find_for_authentication(warden_conditions)
    where(:email => warden_conditions[:email], :subdomain => warden_conditions[:subdomain]).first
  end
end
```

Congrats, User login is now scoped to subdomain!