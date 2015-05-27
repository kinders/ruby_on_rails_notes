add following file app/lib/devise_registrations_controller_decorator.rb

```
  module DeviseRegistrationsControllerDecorator
    extend ActiveSupport::Concern

    included do
      alias :devise_new :new
      def new; custom_new; end
    end
    
    def custom_new
      build_resource({}) # Custom call what you want
      respond_with self.resource
    end
  end

  Devise::RegistrationsController.send(:include, DeviseRegistrationsControllerDecorator)
```

add line into config/environments/development.rb

```
  config.to_prepare do
    Devise::RegistrationsController.send(:include, DeviseRegistrationsControllerDecorator)
  end
```
  
[Source](http://joelazemar.fr/post/65224324836/override-devise-registration-controller)