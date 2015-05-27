# vim: set filetype=markdown :
# Add a role to the User Model 在用户模型中添加角色

## Set up the Role and User models 设置角色和用户模型

First create the Role model and link it to the User Model
首先创建一个Role模型，链接到用户模型中

```
$ rails g model Role name:string
$ rails g migration addRoleIdToUser role_id:integer
$ rake db:migrate
```

Then in your Models
然后在模型中：

```ruby
class User < ActiveRecord::Base
  belongs_to :role
end
class Role < ActiveRecord::Base
  has_many :users
end
```
## Set up seeds.rb with your roles  用角色设置种子

```ruby
['registered', 'banned', 'moderator', 'admin'].each do |role|
  Role.find_or_create_by({name: role})
end
```

Then
然后植入种子：

```
$ rake db:seed
```

## Set up the default role as a model callback  在用户模型中设置默认角色作为回调

```ruby
class User < ActiveRecord::Base
  belongs_to :role
  before_create :set_default_role

  private
  def set_default_role
    self.role ||= Role.find_by_name('registered')
  end
end
```
