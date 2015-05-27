# 认证失败之后重定向到本地化页面

Set your locale and make your default_url_options a **class method** in your ApplicationController
设置本地，在程序控制器中设置default_url_options类方法。

```ruby
    def set_locale
      I18n.locale = params[:locale]
    end

    def self.default_url_options(options={})
      options.merge({ :locale => I18n.locale })
    end
```

_Note:_ this differs from the [Rails Internationalization (I18n) API Guide](http://guides.rubyonrails.org/i18n.html#setting-the-locale-from-the-url-params), but is required by the implementation of the Devise FailureApp (it calls `ApplicationController.default_url_options(*args)` in its default_url_options method).
注意：这不同于《Rails指南·国际化》里《从地址参数设置本地》，但它需要Devise FailureApp的执行。（在default_url_options方法里调用`ApplicationController.default_url_options(*args)`）

Without this change, and if your routes are setup to require a locale, you will see errors like this on authentication failure:
没有这个改变，如果路由设置为要求一个本地，将看到类似下面的认证失败的错误：
`ActionController::UrlGenerationError (No route matches {:action=>"new", :controller=>"devise/sessions"} missing required keys: [:locale])`

