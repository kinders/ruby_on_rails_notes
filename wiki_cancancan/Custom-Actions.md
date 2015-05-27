When you define a user's abilities for a given model, you are not restricted to the 7 RESTful actions (create, update, destroy, etc.), you can create your own.
为给定模型定义用户能力时，可以不限于那7种表转风格的动作，可以创建自己的动作。

For example, in [[Role Based Authorization]] I showed you how to define separate roles for a given user.
比如，在《基于角色的授权》里，展示了怎么为给定的用户定义独立的角色。
However, you don't want all users to be able to assign roles, only admins.
不过，那可不想让所有用户都能分配角色，应该只有管理员才能。
How do you set these fine-grained controls? Well you need to come up with a new action name.
怎么设置这些细致的控制呢？
需要创建一个新的动作。
Let's call it `assign_roles`.
命名为`分配角色`。
 
```ruby
# in models/ability.rb
can :assign_roles, User if user.admin?
```

We can then check if the user has permission to assign roles when displaying the role checkboxes and assigning them.
然后就可以检查用户是否具有需要来分配任务了，如果有就显示角色多选框，并分配角色。

```rhtml
<!-- users/_form.html.erb -->
<% if can? :assign_roles, @user %>
  <!-- role checkboxes go here -->
<% end %>
```

```ruby
# users_controller.rb
def update
  authorize! :assign_roles, @user if params[:user][:assign_roles]
  # ...
end
```

Now only admins will be able to assign roles to users.
现在只有管理员可能分配角色给用户了。

# vim: set filetype=markdown :#
