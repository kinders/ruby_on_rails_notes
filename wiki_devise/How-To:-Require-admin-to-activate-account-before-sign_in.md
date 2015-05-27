# 登录之前要求管理员激活账户

Instead of using the confirmable module (to allow users to activate their own accounts), you may want to require an admin or moderator to activate new accounts.
你可能要求管理员或建模人激活新的账户，而不是使用确认模块。

## Model changes  模型改变

Create a migration as follows (in this case, I'm assuming the model is called User):
像下面那样创建一个迁移（在这个例子中，假定模型是User）：

<kinder:note> 在用户模型增加一个“批准”字段
```sh
  $ rails g migration add_approved_to_user approved:boolean
```

```ruby
class AddApprovedToUser < ActiveRecord::Migration
  def self.up
    add_column :users, :approved, :boolean, :default => false, :null => false
    add_index  :users, :approved
  end

  def self.down
    remove_index  :users, :approved
    remove_column :users, :approved
  end
end
```

Note: You may want to add approved to your 'attr_accessible' in your user model so that you can do bulk assignment when creating new users. 
注意：你可能想添加改进到user模型的`attr_accessible`中，以便在创建新用户时可以大量赋值。
This is deprecated, and incorrect for Rails 4.
这是Rails 4不赞成使用的，也是不正确。
Use [Strong Parameters](http://edgeapi.rubyonrails.org/classes/ActionController/StrongParameters.html) instead.
不如使用健壮参数。
 
Then, override the following methods in your model (User.rb):
然后在user模型中改写下面的方法：

```ruby
  def active_for_authentication?   # 是否激活认证 
    super && approved? 
  end 
  
  def inactive_message # 未激活的信息
    if !approved?   # 是否未批准
      :not_approved 
    else 
      super # Use whatever other message 使用其他信息
    end 
  end
```

You will need to create an entry for `:not_approved` and `:signed_up_but_not_approved` in the i18n file, located at `config/locales/devise.##.yml`:
你将需要在i18n文件为`:not_approved`和`:signed_up_but_not_approved`创建一个条目，位于`config/locales/devise.##.yml`：

```ruby
  devise:
    registrations:
      user:
        signed_up_but_not_approved: 'You have signed up successfully but your account has not been approved by your administrator yet'
    failure:
      not_approved: 'Your account has not been approved by your administrator yet.'
```

## Controllers and Views  控制器和视图

You'll want to create a controller method that is admin-accessible only, that lists the unapproved users and provides a simple way to approve them.
你会想创建一个只有管理员才能访问的控制器方法，列出未经批准的用户，并提供简单的方法来批准它们。

I added a simple link in my index.html.haml page to filter the results to show 'Users awaiting approval'
我在index.html.haml页面添加了一个简单的链接，过滤结果，显示等待批准的用户。

```haml
%h1 Users

= link_to "All Users", :action => "index"
|
= link_to "Users awaiting approval", :action => "index", :approved => "false"

%table
	- @users.each do |user|
		%tr
			%td= user.email
			%td= user.approved
			%td= link_to "Edit", edit_user_path(user)
```

Then in my users controller I have this:
然后在users控制器：

```ruby
  def index
    if params[:approved] == "false"
      @users = User.find_all_by_approved(false)
    else
      @users = User.all
    end
  end
```

## Email Notifications 电邮提示

Add to `app/models/user.rb`
添加到 `app/models/user.rb`：

```ruby
  after_create :send_admin_mail
  def send_admin_mail
    AdminMailer.new_user_waiting_for_approval(self).deliver
  end
```

For more details how to send email, see [ActionMailer](http://guides.rubyonrails.org/action_mailer_basics.html)
发送邮件的细节详见《Rails指南》中的《动作邮箱》一文。

## Reset password instructions  重设密码指令

In the model (app/model/user.rb):
在模型（app/model/user.rb）：

```ruby
  def self.send_reset_password_instructions(attributes={})
    recoverable = find_or_initialize_with_errors(reset_password_keys, attributes, :not_found)
    if !recoverable.approved?
      recoverable.errors[:base] << I18n.t("devise.failure.not_approved")
    elsif recoverable.persisted?
      recoverable.send_reset_password_instructions
    end
    recoverable
  end
```
