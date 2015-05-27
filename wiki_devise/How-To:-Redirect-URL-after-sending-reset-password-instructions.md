Source: http://stackoverflow.com/questions/8809681/cannot-override-devise-passwords-controller

# Look at the source code for devise's PasswordsController:

https://github.com/plataformatec/devise/blob/master/app/controllers/devise/passwords_controller.rb#L42

***

You'll have to create a PasswordsController in your app that inherits from Devise::PasswordsController, implement only the after_sending_reset_password_instructions_path_for(resource_name) method and when setting the routes tell devise to use your controller

***

```ruby
class PasswordsController < Devise::PasswordsController
  protected
  def after_sending_reset_password_instructions_path_for(resource_name)
    #return your path
  end
end
```

***

# in routes

***

```
devise_for :users, :controllers => { :passwords => "passwords" }
```