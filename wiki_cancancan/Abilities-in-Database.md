# vim: set ft=markdown :
# 在数据库里定义能力
What if a non-programmer needs to modify the user abilities, or you want to change them without having to re-deploy the application? 
如果一个无编程能力的人想修改用户能力，或者想要无须重新部署程序就能改变别人的能力呢？
In that case it may be best to store the permission logic in a separate model, let's call it Permission.
那时最好将权限逻辑放在一个独立的模型中，比如叫Permission类。
It is easy to use the database records when defining abilities.
定义能力时，使用数据库记录是很容易的。

For example, let's assume that each user `has_many :permissions`, and each permission has "action", "subject_class" and "subject_id" columns.
比如，假设每个用户`has_many :permissions`，每个权限都有动作、主题类和主题id三栏。
The last of which is optional.
最后一个主题id是可选的。

```ruby
class Ability
  include CanCan::Ability

  def initialize(user)
    can do |action, subject_class, subject|
      user.permissions.find_all_by_action(aliases_for_action(action)).any? do |permission|
        permission.subject_class == subject_class.to_s &&
          (subject.nil? || permission.subject_id.nil? || permission.subject_id == subject.id)
      end
    end
  end
end
```

An alternative approach is to define a separate "can" ability for each permission.
一个可选方法是为每个权限定义一个独立的"can"能力。

```ruby
def initialize(user)
  user.permissions.each do |permission|
    if permission.subject_id.nil?
      can permission.action.to_sym, permission.subject_class.constantize
    else
      can permission.action.to_sym, permission.subject_class.constantize, :id => permission.subject_id
    end
  end
end
```

The actual details will depend largely on your application requirements, but hopefully you can see how it's possible to define permissions in the database and use them with CanCan.
实际的细节取决于程序的需求，但很希望你可以知道，怎么在数据库里定义权限，和CanCan连用。
You can mix-and-match this with defining permissions in the code as well.
也可以在定义权限的代码中混用、匹配下面这些代码。
This way you can keep the more complex logic in the code so you don't need to shoe-horn every kind of permission rule into an overly-abstract database.
这个方法可以保持代码的更复杂的逻辑，所以不需要强制每种权限规则限制在一个过于抽象的数据库中。

You can also create a Permission model containing all possible permissions in your app.
也可以创建一个权限模型，包含程序中所有可能的权限。
Use that code to create a rake task that fills a Permission table:
使用该代码可以创建一个rake任务来完成权限表：

(The code below is not fully tested)
（下面的代码还没得到充分测试）

To use the following code, the permissions table should have such fields :name, :user_id, :subject_class, :subject_id, :action, and :description.
You can generate the permission model by the command: 
要使用下面的代码，权限表需要这些字段：名称，用户id，主题类，主题id，动作，描述。
命令如下：

```ruby
`rails g model Permission user_id:integer name:string subject_class:string subject_id:integer action:string description:text`.
```

```ruby
class ApplicationController < ActionController::Base
  ...
  protected

  # Derive the model name from the controller. egs UsersController will return User
  # 模型名称源自控制器。比如UsersController会返回User
  def self.permission
    return name = controller_name.classify.constantize
  end
end
```

```ruby
def setup_actions_controllers_db

  write_permission("all", "manage", "Everything", "All operations", true)

  controllers = Dir.new("#{Rails.root}/app/controllers").entries
  controllers.each do |controller|
    if controller =~ /_controller/
      foo_bar = controller.camelize.gsub(".rb","").constantize.new
    end
  end
  # You can change ApplicationController for a super-class used by your restricted controllers
  # 可将ApplicationController改为一个由被限制的控制器使用的超级类。
  ApplicationController.subclasses.each do |controller|
    if controller.respond_to?(:permission)	
      clazz, description = controller.permission
      write_permission(clazz, "manage", description, "All operations")
      controller.action_methods.each do |action|
        if action.to_s.index("_callback").nil?
          action_desc, cancan_action = eval_cancan_action(action)
          write_permission(clazz, cancan_action, description, action_desc)
        end
      end
    end
  end
	
end


def eval_cancan_action(action)
  case action.to_s
  when "index", "show", "search"
    cancan_action = "read"
    action_desc = I18n.t :read
  when "create", "new"
    cancan_action = "create"
    action_desc = I18n.t :create
  when "edit", "update"
    cancan_action = "update"
    action_desc = I18n.t :edit
  when "delete", "destroy"
    cancan_action = "delete"
    action_desc = I18n.t :delete
  else
    cancan_action = action.to_s
    action_desc = "Other: " << cancan_action
  end
  return action_desc, cancan_action
end

def write_permission(class_name, cancan_action, name, description, force_id_1 = false)
  permission  = Permission.find(:first, :conditions => ["subject_class = ? and action = ?", class_name, cancan_action]) 
  if not permission
    permission = Permission.new
    permission.id = 1 if force_id_1
    permission.subject_class =  class_name
    permission.action = cancan_action
    permission.name = name
    permission.description = description
    permission.save
  else
    permission.name = name
    permission.description = description
    permission.save
  end
end
```
