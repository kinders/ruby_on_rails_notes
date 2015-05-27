# vim: set ft=markdown :#
# 使用子域名
If you are using subdomains and Devise you may encounter this problem in internet explorer:
如果使用子域名和Devise，在互联网浏览器上会遇到一些问题：

```
Started GET "/" for 127.0.0.1 at 2012-05-30 19:18:51 +1000
Processing by Users::DashboardController#home as HTML
Completed 401 Unauthorized in 1ms
```

You need to configure `config/initializers/session_store.rb` to share across subdomains, for example:
需要配置`config/initializers/session_store.rb`来共享给子域名，比如：

```ruby
MyApp::Application.config.session_store :cookie_store, key: '_my_app_session', domain: '.example.com'
```

To configure per environment you could use the following:
要配置每个环境，需要使用下面的代码：

```ruby
MyApp::Application.config.session_store :cookie_store, key: '_my_app_session', domain: {
  production: '.example.com',
  development: '.example.dev'
}.fetch(Rails.env.to_sym, :all)
```
