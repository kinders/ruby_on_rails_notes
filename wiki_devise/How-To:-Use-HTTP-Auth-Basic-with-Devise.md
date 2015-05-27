# 使用http认证基础
**NOTE**: HTTP Basic authentication is implemented by Devise so the only code required is a call to authenticate_user! in your controller (which will authenticate both login form users and http basic auth users).  
注意：http基本认证通过Devise实现，所以唯一需要的代码是在控制器中调用authenticate_user!（该控制器将认证登录表单用户和http基本认证用户）

See https://github.com/plataformatec/devise/wiki/How-To:-Use-HTTP-Basic-Authentication for instructions.
参见维基《使用http基本认证》

The following is a sample for a Api Controller that will allow http basic and run it through your existing devise configuration.
下面是一个示例，Api控制器允许http基本，通过已有的devise配置来运行。
```ruby
class Api::ApiController < ApplicationController

  before_filter :check_auth


  def check_auth
    authenticate_or_request_with_http_basic do |username,password|
      resource = User.find_by_email(username)
      if resource.valid_password?(password)
        sign_in :user, resource
      end
    end
  end

end
```
