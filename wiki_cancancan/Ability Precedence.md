## 能力的顺序
The ability rules further down in a file will override a previous one.
文件里后面的能力会覆盖前面的能力。
For example, let's say we want the user to be able to do everything to projects except destroy them.
比如，想允许用户拥有对项目除了删除之外的能力。
This is the correct way.
这是正确的做法：

```ruby
can :manage, Project
cannot :destroy, Project
```

It is important that the `cannot :destroy` line comes after the `can :manage` line.
让`cannot :destroy`行位于`can :manage`行之后非常重要。
If they were reversed, `cannot :destroy` would be overridden by `can :manage`.
如果位置颠倒了，`cannot :destroy`将会被`can :manage`覆盖。
Therefore, it is best to place the more generic rules near the top.
因此，最好将更一般化的规则放在顶部。

## `can`的顺序
Adding `can` rules do not override prior rules, but instead are logically or'ed.
添加`can`规则不能改写更高级的规则，在逻辑上的“或”操作。
<kinder:note> 有些令人费解。这是google的翻译。

```ruby
can :manage, Project, :user_id => user.id
can :update, Project do |project|
  !project.locked?
end
```

For the above, `can? :update` will always return true if the `user_id` equals `user.id`, even if the project is locked. 
在上面，如果`user_id`等于`user.id`，`can? :update`将总是返回真，即使项目已经被锁定。

This is also important when dealing with roles which have inherited behavior.
处理继承了该行为的角色时，也很重要。
For example, let's say we have two roles, moderator and admin.
比如，有两个角色，建模人和管理员。
We want the admin to inherit the moderator's behavior.
想让管理员继承建模人的行为。

```ruby
if user.role? :moderator
  can :manage, Project
  cannot :destroy, Project
  can :manage, Comment
end
if user.role? :admin
  can :destroy, Project
end
```

Here it is important the admin role be after the moderator so it can override the `cannot` behavior to give the admin more permissions.
这里重要的是管理员角色在建模人之后，所有管理员可以覆盖`cannot`的表现，具有更多许可。
See [[Role Based Authorization]].
参见《基于角色的授权》

If you are not getting the behavior you expect, please [[post an issue|https://github.com/CanCanCommunity/cancancan/issues]].
如果不能获得期待的表现，请在github里提出问题。

## Additional Docs 其他文档

* [[Defining Abilities]]
* [[Checking Abilities]]
* [[Debugging Abilities]]
* [[Testing Abilities]]
