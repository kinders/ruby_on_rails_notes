In your application controller file **app/controllers/application_controller.rb** include the following private method (below private):
在程序控制器文件中包含下面的私有方法（在private之后）：

```ruby
class ApplicationController < ActionController::Base
  private

  # Overwriting the sign_out redirect path method
  # 改写登出重定向路径的方法
  def after_sign_out_path_for(resource_or_scope)
    root_path
  end
end
```

The return value of this method is the redirect url after sign-out, so you should swap **root_path** to set where devise redirects the users after signing out.
这个方法的返回值是登出之后的重定向地址，所以可以交换根目录来设置devise重定向用户登出之后的路径。
The method is overloading the one contained in [lib/devise/controllers/helpers.rb](https://github.com/plataformatec/devise/blob/master/lib/devise/controllers/helpers.rb) within the gem.
该方法正过载，包含在软件包里的lib/devise/controllers/helpers.rb文件中。


You should also override method [Devise::Controllers::Helpers#stored_location_for](http://rubydoc.info/gems/devise/Devise/Controllers/Helpers#stored_location_for-instance_method) in your application controller, to return nil. 
This applies to after_sign_in_path_for also. YMMV.
也可以在程序控制器中改写方法，返回nil。
这也会应用到after_sign_in_path_for。
