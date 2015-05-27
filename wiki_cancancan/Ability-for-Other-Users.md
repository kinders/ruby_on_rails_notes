What if you want to determine the abilities of a `User` record that is not the `current_user`? 
如果想判断非当前用户的用户的能力，那该怎么做？
Maybe we want to see if another user can update an article.
可能想看看另一个用户是否能更新一个作品。

```ruby
some_user.ability.can? :update, @article
```

You can easily add an `ability` method in the `User` model.
可以在用户模型中定义一个`ability`方法。

```ruby
def ability
  @ability ||= Ability.new(self)
end
```

I also recommend adding delegation so `can?` can be called directly from the user.
也推荐添加委托，这样`can?`就可以从该用户直接调用了。

```ruby
class User < ActiveRecord::Base
  delegate :can?, :cannot?, :to => :ability
  # ...
end

some_user.can? :update, @article
```

Finally, if you're using this approach, it's best to override the `current_ability` method in the `ApplicationController` so it uses the same method.
最后，如果正在使用这个功能，那最好在程序控制器覆盖掉`current_ability`方法，让它使用同样的方法。

```ruby
def current_ability
  current_user.ability
end
```

The downside of this approach is that [[Accessing Request Data]] is not as easy, so it depends on the needs of your application.
这种处理的麻烦是访问请求数据不太容易，所以这取决于所开发程序的需求。

A different approach is taken by [cantango](https://github.com/kristianmandrup/cantango) a gem that builds on top of cancan, and exposes methods such as `#user_can?` and `#admin_can?` for each devise user. 
另一个处理是通过cantango，这个软件包构建于cancan的上层，并将类似`#user_can?`和`#admin_can?`的方法暴露给每个devise用户。
These methods wrap the `#current_[user_type]` methods exposed by devise.
这些方法包装在`#current_[user_type]`里，由devise暴露。
