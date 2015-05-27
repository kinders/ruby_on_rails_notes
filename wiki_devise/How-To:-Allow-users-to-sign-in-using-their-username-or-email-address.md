# vim: set ft=markdown :

# Allow users to Sign In using their username or email address
# 允许用户通过用户名或邮件登录

For this example, we will assume your model is called `User`
在这个例子中，我们假设你的模型叫做`User`。

### Create a username field in the `users` table 创建用户名字段

#### Create a migration: 创建迁移

```ruby
   rails generate migration add_username_to_users username:string:uniq
```

#### Run the migration: 运行迁移

```ruby
   rake db:migrate
```

#### *Rails 3:*

Modify the `User` model and add username, email, password, password confirmation and remember me to `attr_accessible`

```ruby
   attr_accessible :username, :email, :password, :password_confirmation, :remember_me
```

#### *Rails 4:* 修改健壮参数

Modify `application_controller.rb` and add username, email, password, password confirmation and remember me to `configure_permitted_parameters`

```ruby
class ApplicationController < ActionController::Base
  before_action :configure_permitted_parameters, if: :devise_controller?

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.for(:sign_up) { |u| u.permit(:username, :email, :password, :password_confirmation, :remember_me) }
    devise_parameter_sanitizer.for(:sign_in) { |u| u.permit(:login, :username, :email, :password, :remember_me) }
    devise_parameter_sanitizer.for(:account_update) { |u| u.permit(:username, :email, :password, :password_confirmation, :current_password) }
  end
end
```

> see also ["strong parameters"](https://github.com/plataformatec/devise#strong-parameters)


### Create a login virtual attribute in the `User` model  创建登录虚拟属性

#### Add login as an attr_accessor

```ruby
  # Virtual attribute for authenticating by either username or email
  # This is in addition to a real persisted field like 'username'
  attr_accessor :login
```

#### *Rails 3:* Also add login to attr_accessible

```ruby
  attr_accessible :login
```

#### *Rails 4:* same as the first one

```ruby
  attr_accessor :login
```

or if you will use this variable somwhere else in the code:

```ruby
  def login=(login)
    @login = login
  end

  def login
    @login || self.username || self.email
  end
```


### Tell Devise to use :username in the authentication_keys 通知devise

Modify config/initializers/devise.rb to have: 
修改启动器

```ruby
   config.authentication_keys = [ :login ]
```

If you are using multiple models with Devise, it is best to set the authentication_keys on the model itself if the keys may differ:
如果使用多个模型，最后在模型自身设置好认证键。

```ruby
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, 
         :validatable, :authentication_keys => [:login]
```

### Overwrite Devise's `find_for_database_authentication` method in `User` model

Because we want to change the behavior of the login action, we have to overwrite the `find_for_database_authentication` method.
The method's stack works like this: `find_for_database_authentication` calls `find_for_authentication` which calls `find_first_by_auth_conditions`.
 Overriding the `find_for_database_authentication` method allows you to edit database authentication; overriding `find_for_authentication` allows you to redefine authentication at a specific point (such as token, LDAP or database).
Finally, if you override the `find_first_by_auth_conditions` method, you can customize finder methods (such as authentication, account unlocking or password recovery).
因为我们想改变login动作的表现，只能改写`find_for_database_authentication`方法。
方法的堆栈运行类似于：`find_for_database_authentication`调用`find_for_authentication`，由`find_first_by_auth_conditions`。
改写`find_for_database_authentication`方法允许你编辑数据库认证；
改写`find_for_authentication`方法允许你定义特定点（例如密钥、LDAP或数据库）上的认证。
最后，如果改写了`find_first_by_auth_conditions`方法，可以定制finder方法（例如认证、账户解锁或密码改写）


#### For ActiveRecord:

**MySQL users**: the use of the SQL `lower` function below is [most likely unnecessary](https://dev.mysql.com/doc/refman/5.7/en/case-sensitivity.html) and will cause any index on the `email` column to be ignored. 
**MySQL用户**：下面使用SQL的"lower"函数来自《最可能不必要》，会导致任何邮件栏的索引被忽略。

```ruby
# app/models/user.rb

    def self.find_for_database_authentication(warden_conditions)
      conditions = warden_conditions.dup
      if login = conditions.delete(:login)
        where(conditions.to_h).where(["lower(username) = :value OR lower(email) = :value", { :value => login.downcase }]).first
      else
        where(conditions.to_h).first
      end
    end
```

Be sure to add case **insensitivity** to your validations on `:username`:
确保验证`:username`时添加不区分大小写选项。

```ruby
# app/models/user.rb

validates :username,
  :presence => true,
  :uniqueness => {
    :case_sensitive => false
  } # etc.
```

Alternatively, change the find conditions like so:
可选的，像这样修改find条件。

```ruby
  # when allowing distinct User records with, e.g., "username" and "UserName"...
  where(conditions).where(["username = :value OR lower(email) = lower(:value)", { :value => login }]).first
```

#### For Mongoid:

Note: This code for Mongoid does some small things differently than the ActiveRecord code above.  Would be great if someone could port the complete functionality of the ActiveRecord code over to Mongoid [basically you need to port the 'where(conditions)'].  It is not required but will allow greater flexibility.

```ruby
  field :email

  def self.find_first_by_auth_conditions(warden_conditions)
    conditions = warden_conditions.dup
    if login = conditions.delete(:login)
      self.any_of({ :username =>  /^#{Regexp.escape(login)}$/i }, { :email =>  /^#{Regexp.escape(login)}$/i }).first
    else
      super
    end
  end
```

The code below also supports Mongoid but uses the where method and the OR operator to choose between username and email.

```ruby
  # function to handle user's login via email or username
  def self.find_for_database_authentication(warden_conditions)
    conditions = warden_conditions.dup
    if login = conditions.delete(:login).downcase
      where(conditions).where('$or' => [ {:username => /^#{Regexp.escape(login)}$/i}, {:email => /^#{Regexp.escape(login)}$/i} ]).first
    else
      where(conditions).first
    end
  end 
```


### Update your views 更新视图

Make sure you have the Devise views in your project so that you can customize them
确保在项目中有Devise视图，才能自定义它们。

#### *Rails 3 & 4:*

```ruby
   rails g devise:views
```
or
或者

```ruby
   rails g devise:views users
```
in case you have multiple models and you want customized views. 
如果你有多个模型的话。


Simply modify config/initializers/devise.rb file to have
只修改config/initializers/devise.rb文件：

```ruby
   config.scoped_views = true
```

#### *Rails 2:*

```ruby
   script/generate devise_views
```



### Modify the views 修改视图

sessions/new.html.erb:

```diff
  -  <p><%= f.label :email %><br />
  -  <%= f.email_field :email %></p>
  +  <p><%= f.label :login %><br />
  +  <%= f.text_field :login %></p>
```

registrations/new.html.erb:

```diff
  +  <p><%= f.label :username %><br />
  +  <%= f.text_field :username %></p>
     <p><%= f.label :email %><br />
     <%= f.email_field :email %></p>
```

Newer versions have different html.

registrations/edit.html.erb

```diff
  +  <p><%= f.label :username %><br />
  +  <%= f.text_field :username %></p>
     <p><%= f.label :email %><br />
     <%= f.email_field :email %></p>
```

Newer versions have different html.

### Manipulate the `:login` label that Rails will display 国际化错误

*Rails 3 & 4* (config/locales/devise.en.yml)

change
将

```yaml
     invalid: "Invalid email or password."
     ...
     not_found_in_database: "Invalid email or password."
```

to
改为

```yaml
     invalid: "Invalid login or password."
     ...
     not_found_in_database: "Invalid login or password."
```

If instead you see
如果看到的是：

``` yaml
    invalid: "Invalid %{authentication_keys} or password."
    ...
    not_found_in_database: "Invalid %{authentication_keys} or password."
```
you don't have to make any changes.
就不用修改了。


#  Allow users to recover their password or confirm their account using their username 
#  允许用户通过用户名重设密码或验证邮箱

This section assumes you have run through the steps in *Allow users to Sign In using their username or email*.
这一节假设你已经通过了上面的步骤。

#### Configure Devise to use username as reset password or confirmation keys: 配置重设键和确认键

Simply modify config/initializers/devise.rb to have:
修改启动器：

```ruby
   config.reset_password_keys = [ :username ]
   config.confirmation_keys = [ :username ]
```

#### Use `find_first_by_auth_conditions` instead of `find_for_database_authentication` 修改finder

Replace (in your `Users.rb`):
将下面：
```ruby
def self.find_for_database_authentication(warden_conditions)
  conditions = warden_conditions.dup
  if login = conditions.delete(:login)
    where(conditions).where(["lower(username) = :value OR lower(email) = :value", { :value => login.downcase }]).first
  else
    where(conditions).first
  end
end
```

with:
改为：

```ruby
def self.find_first_by_auth_conditions(warden_conditions)
  conditions = warden_conditions.dup
  if login = conditions.delete(:login)
    where(conditions).where(["lower(username) = :value OR lower(email) = :value", { :value => login.downcase }]).first
  else
    if conditions[:username].nil?                    # <kinder:note> 
      where(conditions).first       
    else                                             # <kinder:note> 
      where(username: conditions[:username]).first   #<kinder:note> 
    end                                              #<kinder:note>
  end
end
```
#### Update your views  更新视图

passwords/new.html.erb:

```diff
  -  <p><%= f.label :email %><br />
  -  <%= f.email_field :email %></p>
  +  <p><%= f.label :username %><br />
  +  <%= f.text_field :username %></p>
```

confirmations/new.html.erb:

```diff
  -  <p><%= f.label :email %><br />
  -  <%= f.email_field :email %></p>
  +  <p><%= f.label :username %><br />
  +  <%= f.text_field :username %></p>
```


# Gmail or me.com Style  Gmail风格

Another way to do this is me.com and gmail style.
You allow an email or the username of the email.
允许一个电邮或者电邮的用户名。
For public facing accounts, this has more security.
对于面向公众的账户，这会更安全。
Rather than allow some hacker to enter a username and then just guess the password, they would have no clue what the user's email is.
不再允许黑客输入用户名，然后猜想密码；对于用户的电邮，不再有什么提示。
Just to make it easier on the user for logging in, allow a short form of their email to be used e.g "someone@domain.com" or just "someone" for short.
只让用户登录更容易，允许使用较短形式，比如让someone@domain.com简化成someone。

```ruby
before_create :create_login

  def create_login
    email = self.email.split(/@/)
    login_taken = User.where( :login => email[0]).first
    unless login_taken
      self.login = email[0]
    else	
      self.login = self.email
    end	       
  end

  # You might want to use the self.find_first_by_auth_conditions(warden_conditions) above instead
  # 你可能想使用上面的`self.find_first_by_auth_conditions(warden_conditions)`
  # instead of using this find_for_database_authentication as this one causes problems.
  # 而不是使用这个`find_for_database_authentication`，但这会出现一些问题。
  # def self.find_for_database_authentication(conditions)
  #   self.where(:login => conditions[:email]).first || self.where(:email => conditions[:email]).first
  # end
```
