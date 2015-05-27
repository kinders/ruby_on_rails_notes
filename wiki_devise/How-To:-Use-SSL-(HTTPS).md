# vim: set ft=markdown :#
# https协议
The examples below only show how to make devise views into SSL.
下面的例子只显示了怎么让devise视图变成SSL。
Since Rails uses a cookie for its sessions, it is recommended that the entire website should use SSL for security reasons.
因为Rails使用甜饼来记录会话，出于安全考虑推荐整个网站都使用SSL。

Using the [SSL Requirement plugin](https://github.com/retr0h/ssl_requirement):
使用SSL Requirement插件：

For **Devise 1.0**, one way to do sign_in over SSL is:
对于Devise 1.0，通过SSL登录的方式：

```ruby
# in app/controllers/application_controller.rb

class ApplicationController < ActionController::Base
  include SslRequirement

  ...
end

# in config/environment.rb
config.to_prepare do
  SessionsController.ssl_required :new, :create
  RegistrationsController.ssl_required :new, :create
end 
```

**Devise 1.1** you need to do at the bottom:
到Devise 1.1，需要在底部：

```ruby
Devise::SessionsController.ssl_required :new, :create
```

If the code above just requires ssl on the first request in development, you may need to move the last line to a config.to_prepare block inside config/application.rb or config/environment.rb:
如果上面的代码在开发时第一次请求需要ssl，需要移动最后一行到config/application.rb或config/environment.rb文件的一个`config.to_prepare`代码块。

```ruby
config.to_prepare { SessionsController.ssl_required :new, :create }
```

**Rails 3.1** no longer needs the ssl_requirement gem.
Rails 3.1 不再需要ssl_requirement软件包了。
Just place this in your environment file:
只需要将下面这些代码放在环境文件中：

```ruby
#in config/environments/production.rb
config.to_prepare { Devise::SessionsController.force_ssl }
config.to_prepare { Devise::RegistrationsController.force_ssl }
config.to_prepare { Devise::PasswordsController.force_ssl }
# or your customized controller, extending from Devise
```

And make sure to enable SSL on the server (Nginx, Apache, etc.).
确保开启服务器开启SSL。
If the servers are not configured properly, Rails will not recognize the request as SSL (even if it is), and cause an infinite redirect loop.
如果服务器不能合理配置，Rails不认可请求是SSL（即使它真是SSL），并导致一个无穷的重定向循环。

For Nginx you need to add the following line to your nginx.conf:
对于Nginx，你需要添加下面一行到nginx.conf：

    `proxy_set_header X-FORWARDED-PROTO $scheme;` 

This will forward the protocol to your app (https|http)
这将转发协议到程序中

For Apache you can set:
在Apache，可以这样设置：

    RequestHeader set X-FORWARDED-PROTO 'https'

