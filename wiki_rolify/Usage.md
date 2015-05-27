# vim: set ft=markdown : #
# rolify的用法

## Add a role to a user

1. To define a global role:
定义一个全局角色

    ```ruby
        user = User.find(1)
        user.add_role :admin
    ```
2. To define a role scoped to a resource instance
定一个作用于一个资源示例的角色

    ```ruby
        user = User.find(2)
        user.add_role :moderator, Forum.first
    ```
3. To define a role scoped to a resource class
定义一个作用于一个资源类的角色

    ```ruby
        user = User.find(3)
        user.add_role :moderator, Forum
    ```
4. Starting from rolify 2.1, ``grant`` is a method alias for ``add_role``

    ```ruby
        user = User.find(4)
        user.grant :moderator, Forum.last
    ```

## Role queries  角色查询

1. To check if a user has a global role:

    ```ruby
        user = User.find(1)
        user.add_role :admin # sets a global role
        user.has_role? :admin
        => true
    ```
2. To check if a user has a role scoped to a resource:

    ```ruby
        user = User.find(2)
        user.add_role :moderator, Forum.first # sets a role scoped to a resource
        user.has_role? :moderator, Forum.first
        => true
        user.has_role? :moderator, Forum.last
        => false
        user.has_role? :moderator
        => false # returns false because the role is not global
        user.has_role? :moderator, :any
        => true # returns true because the user has at least one moderator role
    ```
3. To check if a user has a role scoped to a resource class:

    ```ruby
        user = User.find(3)
        user.add_role :moderator, Forum # sets a role scoped to a resource class
        user.has_role? :moderator, Forum
        => true
        user.has_role? :moderator, Forum.first
        => true
        user.has_role? :moderator, Forum.last
        => true
        user.has_role? :moderator
        => false
        user.has_role? :moderator, :any
        => true
    ```
4. A global role has an implicit role for all resources:

    ```ruby
        user = User.find(4)
        user.add_role :moderator # sets a global role
        user.has_role? :moderator, Forum.first
        => true
        user.has_role? :moderator, Forum.last
        => true
        user.has_role? :moderator, Forum
        => true
        user.has_role? :moderator, :any
        => true
    ```

## Dynamic shortcuts 动态快捷键

**To be able to use dynamic shortcuts, you have to [enable it](Configuration#wiki-dynamic_shortcuts) first.**

```ruby
        user = User.find(1)
        user.add_role :admin # sets a global role
        user.is_admin?
        => true
        user.add_role :moderator, Forum.first
        user.is_moderator_of? Forum.last
        => false
```

## Multiple role checking 多角色检查

1. Check if the user has ALL specified roles

    ```ruby
        user = User.find(1)
        user.add_role :admin # sets a global role
        user.add_role :moderator, Forum.first # sets a role scoped to a resource instance
        user.add_role :visitor, Forum # sets a role scoped to a resource class
        user.has_all_roles? :admin, { :name => :moderator, :resource => Forum.first }, { :name => :visitor, :resource => Forum }
        => true
        user.has_all_roles? :admin, { :name => :moderator, :resource => Forum.last }
        => false
        user.has_all_roles? :god, { :name => :visitor, :resource => Forum }
        => false
    ```
2. Check if the user has ANY of the specified role(s)

    ```ruby
        user = User.find(1)
        user.add_role :admin # sets a global role
        user.add_role :moderator, Forum.first # sets a role scoped to a resource
        user.add_role :visitor, Forum # set a role scoped to a resource class
        user.has_any_role? :admin, { :name => :moderator, :resource => Forum.first }, { :name => :visitor, :resource => Forum }
        => true
        user.has_any_role? :admin, { :name => :moderator, :resource => Forum.last }
        => true
        user.has_any_role? :god, { :name => :visitor, :resource => Forum }
        => true
    ```

## Remove a role from a user 从用户处移除角色

1. Remove a global role

    ```ruby
        user = User.find(1)
        user.remove_role :admin
        => true # if user previously had an admin role
    ```
or (in case you get errors with the above method)
    ```ruby
        user = User.find(1)
        user.remove_role "admin"
        => true # if user previously had an admin role
    ```
2. Remove a role scoped to a resource instance

    ```ruby
        user = User.find(2)
        user.remove_role :moderator, Forum.first
        => true # if user previously had a moderator role on Forum.first
    ```
3. Remove a role scoped to a resource class

    ```ruby
        user = User.find(3)
        user.remove_role :moderator, Forum
        => true # if user previously had a moderator role on Forum or any instance of Forum
    ```
4. Starting from rolify 2.1, ``revoke`` is a method alias for ``remove_role``

    ```ruby
        user = User.find(4)
        user.revoke :moderator, Forum.first
    ```

**Please note**:

* Trying to remove a global role where the user has a role with the same name on a resource will remove that scoped role (whatever the scope is)
* Trying to remove a class scoped role where the user has an instance scoped role with the same name on a resource will remove that instance scoped role
* Trying to remove a role scoped to a resource class where the user has a global role won't remove it
* Trying to remove a role scoped to a resource instance where the user has a global role won't remove it

## Finders methods 查找方法

1. Find users having a specific role

    ```ruby
        User.with_role :admin
        # => [ list of users that has admin role ]
    ```

2. Find users having _all_ specified roles. Use a hash for resource scoped role

    ```ruby
        User.with_all_roles :admin, { :name => :moderator, :resource => Forum }
        # => [ list of users that has admin _and_ moderator of forum roles ]
    ```

3. Find users having _any_ specified roles. Use a hash too for resource scoped role

    ```ruby
        User.with_any_role :admin, { :name => :moderator, :resource => Forum.last }
        # => [ list of users that has admin _or_ moderator of the last forum roles ]
    ```

## Resource role queries 资源角色查询

Starting with rolify 3.0, you can search roles on instance level or class level resources.

1. Instance level

    ```ruby
        forum = Forum.first
        forum.roles
        # => [ list of roles that are only bound to a forum instance ]
        forum.applied_roles
        # => [ list of roles bound to a forum instance and to the Forum class ]
    ```
2. Class level

    ```ruby
        Forum.with_role(:admin)
        # => [ list of Forum instances that has the role "admin" bound to it ] 
        Forum.with_role(:admin, current_user)
        # => [ list of Forum instances that has the role "admin" bound to it and belongs to current_user roles ]
        
        Forum.find_roles
        # => [ list of roles that are bound to any Forum instance or to the Forum class ]
        Forum.find_roles(:admin)
        # => [ list of roles that are bound to any Forum instance or to the Forum class with "admin" as a role name ]
        Forum.find_roles(:admin, current_user)
        # => [ list of roles that are bound to any Forum instance or to the Forum class with "admin" as a role name and belongs to current_user roles ]
    ```

Previous: [Configuration](https://github.com/EppO/rolify/wiki/Configuration)
