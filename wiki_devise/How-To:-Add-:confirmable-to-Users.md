If you find yourself needing to introduce confirmable to your user model after the application has already been used for sometime, you will end up marking existing users as unconfirmed in the application.

For example, assume that you have a system full of users and you need to implement email notifications for user accounts. In doing this, existing user accounts will be left unconfirmed thus unable to log in.

Here's how you can introduce confirmable to users while also marking existing users as confirmed.

First, add ``devise :confirmable`` to your models/user.rb

```ruby
devise :registerable, :confirmable
```

Then, do the migration as:

```
rails g migration add_confirmable_to_devise
```

Will generate ``db/migrate/YYYYMMDDxxx_add_confirmable_to_devise.rb``. Add the following to it in order to do the migration.

```ruby
class AddConfirmableToDevise < ActiveRecord::Migration
  # Note: You can't use change, as User.update_all will fail in the down migration
  def up
    add_column :users, :confirmation_token, :string
    add_column :users, :confirmed_at, :datetime
    add_column :users, :confirmation_sent_at, :datetime
    # add_column :users, :unconfirmed_email, :string # Only if using reconfirmable
    add_index :users, :confirmation_token, unique: true
    # User.reset_column_information # Need for some types of updates, but not for update_all.
    # To avoid a short time window between running the migration and updating all existing
    # users as confirmed, do the following
    execute("UPDATE users SET confirmed_at = NOW()")
    # All existing user accounts should be able to log in after this.
  end

  def down
    remove_columns :users, :confirmation_token, :confirmed_at, :confirmation_sent_at
    # remove_columns :users, :unconfirmed_email # Only if using reconfirmable
  end
end
```


You can also Generate view if haven't already

```
rails generate devise:views users
```

Do the migration ``rake db:migrate``

Restart the server. All done! 

If not using reconfirmable, update the configuration in `config/initializers/devise.rb`

```ruby
config.reconfirmable = false
```

### Allowing Unconfirmed Access
If you want to add a "grace period" where unconfirmed users may still login, use the `allow_unconfirmed_access_for` config option (which defaults to 0):  

```ruby
# in Devise Initializer
config.allow_unconfirmed_access_for = 365.days
```

Alternatively, you may want to skip required confirmation all-together:

```ruby
# in User.rb
protected
def confirmation_required?
  false
end
```