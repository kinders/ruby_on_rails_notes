# Frequently asked questions 

* [When I start Rails using server, console, whatever, I get this error: ](https://github.com/EppO/rolify/wiki/FAQ#when-i-start-rails-using-server-console-whatever-i-get-this-error)
* [Is it possible to have two different sets of roles in one application?](https://github.com/EppO/rolify/wiki/FAQ#is-it-possible-to-have-two-different-sets-of-roles-in-one-application-)
* [Why rolify doesn't support ruby 1.8 ?](https://github.com/EppO/rolify/wiki/FAQ#why-rolify-doesnt-support-ruby-18-)
* [Does rolify support Single Table Inheritance (STI) ?](https://github.com/EppO/rolify/wiki/FAQ#does-rolify-support-sti-)
* [Is it possible to use UUID with rolify?](https://github.com/EppO/rolify/wiki/_preview#wiki-is-it-possible-to-use-uuid-with-rolify)

## When I start Rails using server, console, whatever, I get this error: 

```console
undefined local variable or method `rolify' for User:Class
```
If you are using Mongoid ORM, make sure you load rolify (using the ``rolify`` method) **after** ``Mongoid::Document`` include in the ``User`` class.
For example:

```ruby
class User
  include Mongoid::Document
  rolify
end
```
If you are using ActiveRecord, you should not get this error, please [fill a bug](https://github.com/EppO/rolify/issues/new)

***
```console
method_missing': undefined local variable or method scopify' for #Class:0x007ff921536890 (NameError)
```

If you get this error, you're probably using ``less-rails`` gem, there is a conflict in the less-rails railtie with rolify's one. In the meantime, add this to your ``application.rb``:
```ruby
require 'rolify/railtie'
```

---
## Is it possible to have two different sets of roles in one application?

If you have two different ``User`` classes, let's say ``User`` and ``AdminUser``, yes it is.
To make it work, you'll also need two ``Role`` classes (``Role`` and ``AdminRole``), and therefore two tables in the database (and two migration files if you're using ActiveRecord). To make it easier, just run the generator twice:

  * ``rails g rolify Role User``
  * ``rails g rolify AdminRole AdminUser``

In your ``User`` model, add ``rolify`` as usual, since ``Role`` is the default rolify Role class
In your ``AdminUser`` model, add ``rolify :role_cname => "AdminRole"`` to specify your alternate Role class

You may wish to give a class roles while using it as a scope for another class' roles.

For example, you might have a ``User`` class with a rolify ``Role`` table and an ``Account`` class with a rolify ``Plan`` table. An ``User`` might have the role ``:admin`` but only scoped to that user's ``Account``, while the account in turn has the role ``:platinum``.

In this case you will have to call ``resourcify`` and ``rolify`` on the same model, ``Account``. Here the order of the calls in your model is significant! This order will work:

```ruby
resourcify
rolify role_cname: "Plan"
```

However, this order will not:

```ruby
rolify role_cname: "Plan"
resourcify
```

Finally, if you want to use a class as a resource for a rolify class called anything but the default, you'll need to specify it in your call:

```ruby
resourcify :plans, role_cname: 'Plan'
```

---
## Why doesn't rolify support ruby 1.8 ?

Mongoid >= 2.0 requires ruby 1.9, that's why rolify dropped ruby 1.8 support to maintain the dependency. Moreover, upcoming Rails 4.0 will drop ruby 1.8 support too. Although rolify doesn't necessarily use ruby 1.9 specific features or syntax, I quit enforcing the compatibility for these 2 reasons.
In the case you _have_ to use 1.8 for whatever reason, this [repository](https://github.com/softcraft-development/rolify/tree/e2eaa7af682583a9452c51a2690e5d82b84fd79d) hosts a fork of rolify supporting ruby 1.8. Don't expect it to be maintained heavily tho.

---
## Does rolify support STI ?

If you use ActiveRecord STI and add ``rolify`` method only in the superclass, you should use ``becomes`` AR method before using any rolify commands, like this:
```ruby
class User < ActiveRecord::Base
  rolify
  ...
end

class UserTypeOne < User
  ....
end

class UserTypeTwo < User
  ....
end

user = UserTypeTwo.find(1)
base_user = user.becomes(User)
base_user.add_role :admin 
```

## Is it possible to use UUID with rolify?
Sure! You just need to fix your migration file. In example below - we have models Role and User:

```
class RolifyCreateRoles < ActiveRecord::Migration
  def change
    create_table :roles, id: :uuid do |t|
      t.string :name
      t.uuid :resource_id
      t.string :resource_type

      t.timestamps
    end

    create_table(:users_roles, :id => false) do |t|
      t.uuid :user_id
      t.uuid :role_id
    end

    add_index(:roles, :name)
    add_index(:roles, [ :name, :resource_type, :resource_id ])
    add_index(:users_roles, [ :user_id, :role_id ])
  end
end
```