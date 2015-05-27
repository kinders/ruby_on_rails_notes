The following migration can be used to add devise required columns to an existing users table.

```ruby
class AddDeviseColumnsToUser < ActiveRecord::Migration
  def change
    change_table :users do |t|
      # if you already have a email column, you have to comment the below line and add the :encrypted_password column manually (see devise/schema.rb).
      t.database_authenticatable
      t.confirmable
      t.recoverable
      t.rememberable
      t.trackable
      t.token_authenticatable
      t.timestamps
    end
  end
end
```

Note: If your existing users table already has a column `email`, then Devise's `t.database_authenticatable` will fail, because it tries to add `email` again. `t.database_authenticatable` will add two columns, `email` and `encrypted_password`. If `email` already exists, then just manually add the `encrypted_password`, in line of `t.database_authenticatable`:

```
  t.string :encrypted_password, :null => false, :default => '', :limit => 128
```

### Devise 2.0 does not provide the migrations helpers 
You can see the list of columns generated automatically by Devise here:

https://github.com/plataformatec/devise/wiki/How-To:-Upgrade-to-Devise-2.0-migration-schema-style

Note: it is sometimes easier to just delete the table and re-create it using Devise instead of trying to retrofit Devise into your table. 

### Adding devise to existing model in the 2.0+ migration schema
You will still have errors in your migration with the 2.0 schema, if you have existing fields such as email.

To eliminate migration errors on duplicate fields, use `t.change` as shown below.

```
t.change :email, :string,     :null => false, :default => "" 
```

Notice that for `t.change` to work, you have to specify the type for the field being changed. In the case of the email migration above, the email field was of type string.