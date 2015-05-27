By default, CanCan works best when the administration functionality is in the same controller as the public site.
默认情况下，CanCan在管理功能和公共站点位于同一个控制器时，运行得最好。
This way each resource only has one controller and set of actions it needs to handle permissions on.
这样，每个资源只有一个控制器和一组它要处理授权的动作。
 
However some sites work better when the administration section has its own separate set of controllers inside an `Admin` namespace such as `Admin::ArticlesController`.
不过有一些站点，它的管理部分在`Admin`命令空间拥有自己的独立控制器，例如`Admin::ArticlesController`时，会运行得更好。
This means there are two controllers handling the same resource.
这意味着有两个控制器处理同一个资源。
Each controller will likely need different permission logic.
每个控制器很可能需要不同的许可逻辑。
The problem is that the `Ability` model knows nothing about the `Admin` namespace.
问题是那个`Ability`模型并不知道`Admin`命名空间。
 
If you go with an `Admin` namespace I recommend creating a base controller which all admin controllers inherit from.
如果设置了`Admin`命令空间，我推荐创建一个base控制器，所有的admin控制器都继承于它。
This is often a good thing to do with any namespace because it can help refactor out common functionality that exists in the controllers in that namespace.
使用命名空间通常是个好事，因为这能帮助重修该命令空间下的控制器已有的普通功能。
 
```ruby
# in controllers/admin/base_controller.rb
module Admin
  class BaseController < ApplicationController
    # common behavior goes here ...
  end
end
```

Then use this as the superclass for each of those controllers.
然后使用这个作为每个控制器的超类。

```ruby
# in controllers/admin/articles_controller.rb
module Admin
  class ArticlesController < BaseController
    # ...
  end
end
```

Now several possibilities open up, and the one to go with depends on how complex the authorization needs to be in this admin namespace.
现在有几种可能了，至于结果怎样，就取决于这个管理命令空间里的授权需求有多复杂了。
If all you need to do is verify `current_user.admin?` then you may want to skip CanCan altogether and use a simple before filter.
如果所有的需求是验证`current_user.admin?`，那么可能会想跳过所有的CanCan，使用一个简单的前置过滤器。
 
```ruby
# in controllers/admin/base_controller.rb
before_filter :verify_admin
private
def verify_admin
  redirect_to root_url unless current_user.try(:admin?)
end
```

There's really no need to use CanCan here because all the behavior is the same.
这里实在没有必要使用CanCan，因为所有的表现都是一样的。
However if you have various levels of admins which needs to access different things then that's a reason to use CanCan.
不过，如果有不同级别的管理，需要访问不同的东西，那还是使用CanCan吧。
In that case I recommend creating a separate `Ability` class.
那种情况下推荐创建独立的`Ability`类。
 
```ruby
# in models/admin_ability.rb
class AdminAbility
  include CanCan::Ability
  def initialize(user)
    # define admin abilities here ....
  end
end
```

Then use this `AdminAbility` for admin controllers.
然后为admin控制器使用这个`AdminAbility`。

```ruby
# in admin/base_controller.rb
def current_ability
  @current_ability ||= AdminAbility.new(current_user)
end
```

Now all admin permission logic will be handled in its own `AdminAbility` class and everything else will be in the normal `Ability` class.
现在所有的管理许可逻辑都在自身的`AdminAbility`类中，其他任何东西都在正常的`Ability`类中。

