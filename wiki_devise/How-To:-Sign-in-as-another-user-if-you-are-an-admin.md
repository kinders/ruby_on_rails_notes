# vim: set ft=markdown :#
# 管理员登录时作为另一个用户

This is a useful administration feature I've seen on a few apps where you can quickly "become" one of the users to see what their profile and screens look like.
这是一个有用的管理特性，在程序中可以快速变成一个用户，看看用户配置和屏幕的模样。

The implementation is simple but it took me a while to find out how to do it, so I'm recording it here:
实现很简单，但我要花一些时间才能知道怎么做到的，记录如下;

```ruby
class AdminController < ApplicationController
  before_filter :authenticate_user!
  
  def become
    return unless current_user.is_an_admin?
    sign_in(:user, User.find(params[:id]))
    redirect_to root_url # or user_root_url
  end
end
```

If you want to ensure that `last_sign_in_at` and `current_sign_in` aren't upda#ted when becoming the user, you can replace `sign_in(:user, User.find(params[:id]))` with `sign_in(:user, User.find(params[:id]), { :bypass => true })` in the example above. 
The `:bypass` option bypasses warden callbacks and stores the user straight in the session. 
如果想在变成用户时要确保`last_sign_in_at`和`current_sign_in`不被更新，可以用`sign_in(:user, User.find(params[:id]), { :bypass => true })`替换`sign_in(:user, User.find(params[:id]))`。
`:bypass`选项会绕过warden回调，并直接将用户保存在会话钟。

For a slightly more advanced & customizable implementation of this concept, packaged as a gem, check out flyerhzm's [[switch_user|https://github.com/flyerhzm/switch_user]] project. 
这个概念的一个更高级、定制性能更强的实现，已经打包成gem，参见flyerhzm的switch_user项目。
