## Using rolify's generator

1. Generate Role model
    First, create your Role model and migration file using this generator:
    
        rails g rolify:role ROLE USER

    If you omit ROLE and/or USER, rolify will use the default classes Role and User. Depending of your needs, you can specify any ROLE class name you want. For the USER class name, you should probably use the one provided by your authentication solution, rolify just adds a class methods inside existing USER class.

    Starting from rolify 3.0, if you want to use Mongoid instead of ActiveRecord, put it as third argument of the generator:

        rails g rolify:role Role User mongoid

    In that case, you need to explicitly put the Role and User classes as first and second arguments.

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