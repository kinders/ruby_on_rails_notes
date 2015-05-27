If your application requires that signed in users access your site using HTTPS protocol, but you need them to be redirected back to HTTP protocol when they sign out, put this in your application_controller.rb file:
如果程序要求登录用户使用https协议访问站点，但登出时重定向到http协议，可将这些代码放在程序控制器中:

    def after_sign_out_path_for(resource_or_scope)
      root_url(:protocol => 'http')
    end

If you want them to be re-directed back to some other address, use that instead of root, but you must use the `*_url` suffix because the `*_path` helpers ignore the :protocol key.
如果想它们被重定向会同样的地址，那就别使用根地址，应该使用那个地址，但必须使用`*_url`后缀，因为`*_path`辅助器会忽略那个协议键。

However, if you have multiple roles, like an admin and client and supervisor, it's a little more complicated, not much:
不过，如果有多个角色，比如管理员、客户和超级用户，那也问题不大：

    def after_sign_out_path_for(resource_or_scope)
      [current_supervisor, current_admin, current_client].compact.count <= 1 ? root_url(:protocol => 'http') : root_path
    end

When this method is called, `current_{resource_name}` for the role from which the user has just signed out is non-nil.
当这个方法被调用时，尽管用户已经登出，该角色的`current_{resource_name}`不是nil。
So you have to check for only 1 role still non-nil, then you know that they are signing out from their last role.
所以必须检查一个非nil的角色，才知道他们已经从最后的角色中登出了。
You see, Devise is so awesome that a user can be signed in under more than one role!
你看，Devise真是可怕，用户可以用多个角色登录。
