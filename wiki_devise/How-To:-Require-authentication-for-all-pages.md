# 所有页面都需要认证
In case your app requires users to be authenticated for all pages and to avoid the "You need to sign in or sign up before continuing." message when trying to access the application root, add this to your routes.rb:
如果程序需要用户认证，用户尝试访问程序根目录时，避免提供“继续之前需要登录或注册”，需要将这一行添加到路由中：

```ruby
authenticated :user do
  root :to => 'home#index', :as => :authenticated_root
end
root :to => redirect('/users/sign_in')
```

#### Route Naming Note 路由命名注意：
As of Rails 4+, two routes with the same **name** will raise an exception at app load time.
到了Rails 4，两个路由都具有同一个名称将会在加载时抛出异常。
In the above example, it is important to note that the two routes respond to the same URL but they have two different route names (`authenticated_root_path` and `root_path`).
上面的例子中，注意：两个路由对应着相同的URL，但有不同的名称（`authenticated_root_path`和`root_path`）。
 
