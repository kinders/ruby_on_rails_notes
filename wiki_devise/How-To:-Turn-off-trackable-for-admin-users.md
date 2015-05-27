Turn off trackable for admin users:
关闭管理员用户的跟踪：

```ruby
class User < ActiveRecord::Base
  def update_tracked_fields!(request)
    super(request) unless admin?
  end
end
```
