You can use CanCan with controllers that do not follow the traditional show/new/edit/destroy actions, however you should not use the `load_and_authorize_resource` method since there is no resource to load.
如果控制器没有传统的“显示”、“新建”、“编辑”、“删除”动作，还是可以使用CanCan；不过就不能使用`load_and_authorize_resource`方法了，因为已经没有资源来加载了。
Instead you can call `authorize!` in each action separately.
相反，可以在每个动作里调用`authorize!`。
 
**NOTE:** This is **not** the same as having additional non-RESTful actions on a RESTful controller. 
**注意**这不同于在表转风格控制器内使用非表转风格的动作。
See the Choosing Actions section of the [[Authorizing Controller Actions]] page for details.
更多信息参见《控制器动作授权》中《选择动作》一节。

For example, let's say we have a controller which does some miscellaneous administration tasks such as rolling log files.
比如，有个控制器，它会处理杂项管理任务，比如轮转日志文件。
We can use the `authorize!` method here.
可以使用`authorize!`方法：
 
```ruby
class AdminController < ActionController::Base
  def roll_logs
    authorize! :roll, :logs
    # roll the logs here
  end
end
```

And then authorize that in the `Ability` class.
然后在`Ability`类中授权。
```ruby
can :roll, :logs if user.admin?
```

Notice you can pass a symbol as the second argument to both `authorize!` and `can`.
注意，可以传入一个符号作为第二个参数给`authorize!`和`can`。
It doesn't have to be a model class or instance.
它不是模型类或实例。
Generally the first argument is the "action" one is trying to perform and the second argument is the "subject" the action is being performed on.
一般第一个参数是想要执行授权的动作，第二个参数是目标动作的主题。
It can be anything.
可以是任何东西。
 
## Alternative: authorize_resource 可选：`authorize_resource`

Alternatively you can use the `authorize_resource` and specify that there's no class.
可选地，可以使用`authorize_resource`并指出它没有类。
This way it will pass the resource symbol instead.
这个方法将会传入资源符号。
This is good if you still have a Resource-like controller but no model class backing it.
如果有个类似资源的控制器、同时背后没有模型类，那就是个好主意了。
 
```ruby
class ToolsController < ApplicationController
  authorize_resource :class => false
  def show
    # automatically calls authorize!(:show, :tool)
  end
end
```
