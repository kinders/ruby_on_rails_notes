Add this to your ApplicationController
将它添加到程序控制器中

```ruby
class ApplicationController < ActionController::Base
  
  protected

  def after_sign_in_path_for(resource)
    # return the path based on resource
    # 返回基于资源的路径
  end

end
```
