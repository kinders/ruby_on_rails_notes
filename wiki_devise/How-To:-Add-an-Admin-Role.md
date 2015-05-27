# 添加管理角色

## Option 1 - Creating an admin model  创建一个管理模型

First generate the model and migration for the Admin role:
首先生成模型和迁移Admin角色：

```sh
$ rails generate devise Admin
```

Configure your Admin model
配置模型：

```ruby
class Admin < ActiveRecord::Base
  devise :database_authenticatable, :trackable, :timeoutable, :lockable  
end
```

Adjust the generated migration:
调整生成的迁移：

Devise 2.2:
```ruby
class DeviseCreateAdmins < ActiveRecord::Migration
  def self.up
    create_table(:admins) do |t|
      t.string :email,              :null => false, :default => ""
      t.string :encrypted_password, :null => false, :default => ""
      t.integer  :sign_in_count, :default => 0
      t.datetime :current_sign_in_at
      t.datetime :last_sign_in_at
      t.string   :current_sign_in_ip
      t.string   :last_sign_in_ip
      t.integer  :failed_attempts, :default => 0 # Only if lock strategy is :failed_attempts
      t.string   :unlock_token # Only if unlock strategy is :email or :both
      t.datetime :locked_at
      t.timestamps
    end
  end

  def self.down
    drop_table :admins
  end
end
```

Earlier Devise Versions:
之前的Devise版本：

```ruby
class DeviseCreateAdmins < ActiveRecord::Migration
  def self.up
    create_table(:admins) do |t|
      t.database_authenticatable :null => false
      t.trackable
      t.lockable
      t.timestamps
    end
  end

  def self.down
    drop_table :admins
  end
end
```

Routes should be automatically updated to contain the following
路由应自动升级来包含下面的代码：

```ruby
 devise_for :admins
```

Enjoy!

## Option 2 - Adding an admin attribute

The easiest way of supporting an admin role is to simply add an attribute that can be used to identify administrators.
支持管理角色的最简单的方法增加一个属性，用来标示管理员。

```sh
$ rails generate migration add_admin_to_users admin:boolean
```

Add this to the migration:
下面是迁移：

```ruby
class AddAdminToUsers < ActiveRecord::Migration
  def self.up
    add_column :users, :admin, :boolean, :default => false
  end

  def self.down
    remove_column :users, :admin
  end
end
```

Your migration will now look like this:
迁移现在看起来是这样：

```ruby
class AddAdminToUsers < ActiveRecord::Migration
  def change
    add_column :users, :admin, :boolean
  end

  class AddAdminToUsers < ActiveRecord::Migration
	  def self.up
	    add_column :users, :admin, :boolean, :default => false
	  end

	  def self.down
	    remove_column :users, :admin
	  end
  end
end
```

Next, execute the migration script:
下面，执行迁移：

```sh
$ rake db:migrate
```

Now you're able to identify administrators:
现在可以标识管理员了：

```ruby
if current_user.admin?
  # do something
end
```

If the page could potentially not have a `current_user` set then:
如果页面没有当前用户，可以设置：

```ruby
if current_user.try(:admin?)
  # do something
end
```

With the above way if `current_user` were `nil`, then it would still work without raising an ``undefined method `admin?' for nil:NilClass`` exception.
如果当前用户为nil，程序依然可以运行，不会抛出一个``undefined method `admin?' for nil:NilClass``异常。

The code below can be used to grant admin status to the current user.
下面的代码可用于将管理权限赋予当前用户。

```ruby
current_user.update_attribute :admin, true
```
