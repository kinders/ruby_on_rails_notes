The default operation for CanCan is to authorize based only on user and the object identified in `load_resource`.  So if you have a `WidgetsController` and also an `Admin::WidgetsController`, it looks to me like CanCan will allow the same authorizations for each.  This probably defeats the whole purpose of creating a separate Admin controller in the first place.

Just like in the example given for [[Accessing Request Data]], you **can** also create differing authorization rules that depend on the controller namespace.  

In this case, just override the `current_ability` method in `ApplicationController` to include the controller namespace, and create an `Ability` class that knows what to do with it.

``` ruby
class ApplicationController < ActionController::Base
  #...

  private

  def current_ability
    # I am sure there is a slicker way to capture the controller namespace
    controller_name_segments = params[:controller].split('/')
    controller_name_segments.pop
    controller_namespace = controller_name_segments.join('/').camelize
    Ability.new(current_user, controller_namespace)
  end
end


class Ability
  include CanCan::Ability

  def initialize(user, controller_namespace)
    case controller_namespace
      when 'Admin'
        can :manage, :all if user.has_role? 'admin'
      else
        # rules for non-admin controllers here
    end
  end
end
```

I used the following code on my ApplicationController: 

``` ruby
# ...
private

def namespace
  # 2012.3.13 didn't work on Rails 3.0.7, cancan 1.6.7; looks promising, but needs some figuring out.
  #cns = @controller.class.to_s.split('::')
  #cns.size == 2 ? cns.shift.downcase : ""

    # I am sure there is a slicker way to capture the controller namespace
    # 2012.3.13 But it works!
    controller_name_segments = params[:controller].split('/')
    controller_name_segments.pop
    controller_namespace = controller_name_segments.join('/').camelize
end

def current_ability
  # Ability.new(current_user, namespace)
  @current_ability ||= Ability.new(current_user, namespace)
end
```

I have not tested it extensively yet.