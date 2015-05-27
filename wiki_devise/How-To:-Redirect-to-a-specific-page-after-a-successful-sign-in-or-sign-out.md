You can override the default behaviour by creating an `after_sign_in_path_for` [[RDoc](http://rubydoc.info/github/plataformatec/devise/master/Devise/Controllers/Helpers:after_sign_in_path_for)] method in your ApplicationController and have it return the path for the page you want:
可以在程序控制器中通过创建一个`after_sign_in_path_for`方法来改写默认行为，它会返回你想要的页面。

```ruby
def after_sign_in_path_for(resource)
 current_user_path
end
```

There exists a similar method for sign out; `after_sign_out_path_for` [[RDoc](http://rubydoc.info/github/plataformatec/devise/master/Devise/Controllers/Helpers:after_sign_out_path_for)]
登出也存在一个类似的方法：`after_sign_out_path_for`

## Keeping user on the same page after signing out 用户登出后页面保持不变

```ruby
def after_sign_out_path_for(resource_or_scope)
  request.referrer
end
```
