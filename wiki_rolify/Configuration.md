**For rolify <= 3.2, [read this](rolify-3.2-and-older)**

## Using rolify's generator _(rolify 3.3 and earlier)_

1. Generate Role model
    First, create your Role model and migration file using this generator:
    
        rails g rolify ROLE USER

    ROLE argument is mandatory, USER is optional. If you omit USER, rolify will use the default class ``User``. Depending of your needs, you can specify any ROLE class name you want. For the USER class name, you should probably use the one provided by your authentication solution, rolify just adds a class method inside an existing USER class.

    If you want to use Mongoid instead of ActiveRecord, put it as third argument of the generator:

        rails g rolify Role User -o mongoid

    By default, ActiveRecord adapter is used.

    **If you upgraded from 3.0 and you use Mongoid, run again the generator to have the new version of the ``Role`` class.** including the indexes.

2. Run the migration (only if you use ActiveRecord adapter, otherwise skip this step)

    Let’s migrate !

        rake db:migrate

3. <p id="dynamic_shortcuts">If you want to use dynamic shortcut methods, uncomment the line in ``config/initializers/rolify.rb``</p> 

  ```ruby
  # Dynamic shortcuts for User class (user.is_admin? like methods). Default is: false
  # Enable this feature _after_ running rake db:migrate as it relies on the roles table
  config.use_dynamic_shortcuts
  ```

  **Starting from rolify v2, dynamic shortcuts are disabled by default.**

## Configure your user model

This gem adds the rolify method to your User class. You can also specify optional callbacks* on the user for when roles are added or removed:
  
  ```ruby
  class User < ActiveRecord::Base
    rolify :role_cname => 'Class', :before_add => :before_add_method

    def before_add_method(role)
      # do something before it gets added
    end
  end
  ```
You'll need to specify ``:role_cname`` if you created another class to handle roles instead of the default one (Role). The same goes to ``resourcify`` method.

The rolify method accepts the following callback options:

* ``before_add``
* ``after_add``
* ``before_remove``
* ``after_remove``

*PLEASE NOTE: when defining callback methods you must specify role argument, otherwise you'll get ``ArgumentError: wrong number of arguments (1 for 0)``. Callbacks are currently only supported using ActiveRecord ORM. Mongoid will support association callbacks in its 3.1 release (Mongoid current version is 3.0.x)

## Configure your resource models

In all the resource models you want to apply roles on, add ``resourcify`` method to the class to provide resource role queries.

For example, for an ``ActiveRecord`` class, just add:

```ruby
class Forum < ActiveRecord::Base
  resourcify :role_cname => 'Class'
end
```

You'll need to specify ``:role_cname`` if you created another class to handle roles instead of the default one (Role). The same goes to ``rolify`` method.

For a ``Mongoid`` class, add it after ``Mongoid::Document`` include:

```ruby
class Forum
  include Mongoid::Document
  resourcify
end
```

Previous: [Installation](https://github.com/EppO/rolify/wiki/Installation) | Next: [Usage](https://github.com/EppO/rolify/wiki/Usage)