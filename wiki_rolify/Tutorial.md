# vim: set ft=markdown :#
# Devise + CanCan + rolify Tutorial

This Tutorial shows you how to setup a [Rails >=3.1](http://rubyonrails.org/) application with a strong and flexible authentication/authorization stack using [Devise](https://github.com/plataformatec/devise), [CanCan](https://github.com/ryanb/cancan) and [rolify](https://github.com/EppO/rolify) (3.0 and later)
这个指南向你展示了怎么设置一个程序，它拥有健壮而有弹性的认证/授权栈堆，使用Devise、CanCan和roliry。

# Installation 安装

1.  First, create a bare new rails app.
    首先，创建一个rails app。
If you already have an existing app with Devise and Cancan set up and you just want to add rolify, just add rolify in your Gemfile, run `bundle install` and skip to step 6 * `# rails new rolify_tutorial` * edit the `Gemfile` and add Devise, CanCan and rolify gems:
如果已有一个安装了Devise和CanCan的程序，则只需在Gemfile中新添rolify，运行`bundle install`，跳过下面的步骤6：

```ruby
gem 'devise'
gem 'cancan'
gem 'rolify'
```

2. run `bundle install` to install all required gems
   运行`bundle install`安装所有需要的软件包

3. Run Devise generator
   运行Devise生成器

  * `# rails generate devise:install`

4. Create the User model from Devise
   从Devise中生成User模型：

  * `# rails generate devise User`

5. Create the Ability class from CanCan
   从CanCan中创建Ability类：

  * `# rails generate cancan:ability`

6. Create the Role class from rolify
   从rolify中创建Role类：

  * `# rails generate rolify Role User`

7. Run migrations
   运行迁移：

  * `# rake db:migrate`

# Configuration 配置

1. Configure Devise according to your needs. 
   根据你的需要配置Devise。
Follow [Devise README](https://github.com/plataformatec/devise/blob/master/README.md) for details.
记得跟着文档的细节做。

2. Edit the Ability model class, add these lines in the initialize method:
  编辑Ability模型类，在initialize方法中添加这些行：

```ruby
if user.has_role? :admin
  can :manage, :all
else
  can :read, :all
end
```

3. Use the resourcify method in all models you want to put a role on. 
   在所有想要进行角色区分的模型中使用resourcify方法：
For example, if we have the Forum model:
比如，如果有一个论坛模型：

```ruby
class Forum < ActiveRecord::Base
  resourcify
end
```

# Usage  用法

1. Create a User using `rails console`
   用rails控制台创建一个用户

```ruby
> user = User.new
> user.email = "anyemail@ddress.com"
> user.password = "test1234"
> user.save
```

2. Add a role to the new User
   给该用户添加一个角色

```ruby
> user.add_role "admin"
```

3. Check if the user has admin rights
   检查该用户是否有管理功能

```ruby
> ability = Ability.new(user)
> ability.can? :manage, :all
  => true
```

# Advanced Usage 高级用法

If you want to use class scoped role with CanCan, it's a bit tricky.
如果想用CanCan的类作用域role，就会有些难解。
Currently in CanCan 1.x, you cannot  mix instance and class checking, because the two are OR-ed and class checking skips the hash of conditions (see [https://github.com/ryanb/cancan/wiki/Checking-Abilities](https://github.com/ryanb/cancan/wiki/Checking-Abilities) for more details).
在当前的CanCan 1.x，不能混合实例检查和类检查，因为两种是或然性的，类检查将会跳过条件化散列。
 Let's take this ability class example:
看看下面的这个ability类的例子：

```ruby
if user.has_role? :admin
  can :manage, :all
else
  can :read, Forum
  can :write, Forum if user.has_role?(:moderator, Forum)
  can :write, Forum, :id => Forum.with_role(:moderator, user).pluck(:id)
end
```

This **won't** work as you expect, because the last ``:write`` clause will always return ``true`` if you ask ``ability.can? :write, Forum``, even if your user has only a role an on instance of Forum.
这是无法如期运行的，因为最后的`write`子句总是返回true，如果你询问`ability.can? :write, Forum`的话，即使用户只拥有论坛实例的角色。
But you can use some workarounds: 
不过可以使用一些工作区：

* don't use the class scoped role for an instance.
  不要为一个实例使用类作用域的角色
That means, you will need class scoped only roles and instance scoped only roles separated.
那意味着，将需要类作用域的角色和实例作用域的角色区分开来。
In that case, it's better to use different action names like: 
那样，最好是使用不同的动作名称，就像：

```ruby
if user.has_role? :admin
  can :manage, :all
else
  can :read, Forum
  can :manage, Forum if user.has_role?(:manager, Forum)
  can :write, Forum, :id => Forum.with_role(:moderator, user).pluck(:id)
end
```

* don't use ``can?`` method for class checking.
  不要使用`can?`方法进行类检查。
Use rolify instead.
应该使用rolify。
so if you want to display a button or some info only if a user has a specific class role, use this: 
所以，如果想展示一个只有用户拥有特定类角色时才出现的按键或信息，使用这个代码：

```rhtml
<% if user.has_role? :moderator %>
  ...
<% end %>
```

and for the instance scoped roles, you still are able to use the ``Ability`` class:
至于实例作用域的角色，仍可以使用`Ability`类：

```ruby
if user.has_role? :admin
  can :manage, :all
else
  can :read, Forum
  can :write, Forum, :id => Forum.with_role(:moderator, user).pluck(:id)
end
```

Please note that the ``with_role`` method allows us to restrict the Forum instances the user has a role on. 
请注意`with_role`方法允许限制为用户拥有角色的论坛实例
It's provided by rolify library using ``resourcify`` method on the ``Forum`` class.
这是论坛类里的resourcify方法时，rolify库提供的。

