You can bypass Cancan 2.0's authorization for Devise controllers similar to Cancan 1.6:
可以绕过Cancan 2.0为Devise控制器的授权，类似与Cancan 1.6：

```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery
  enable_authorization :unless => :devise_controller?
end
```

It may be a good idea to specify the rescue from action:
特定动作的救援，是个好主意。

```ruby
rescue_from CanCan::Unauthorized do |exception|
    if current_user.nil?
      session[:next] = request.fullpath
      puts session[:next]
      redirect_to login_url, :alert => "You have to log in to continue."
    else
      #render :file => "#{Rails.root}/public/403.html", :status => 403
      if request.env["HTTP_REFERER"].present?
        redirect_to :back, :alert => exception.message
      else
        redirect_to root_url, :alert => exception.message
      end
    end
  end
```
