# 创建一个游客账户

In some applications, it's useful to have a guest `User` object to pass around even before the (human) user has registered or logged in.
在一些程序，在用户注册或登录之前有一个匿名的用户对象，是很有帮助的。
 Normally, you want this guest user to persist as long as the browser session persists.
一般，希望这个游客用户和浏览器会话一致。
 
Our approach is to create a guest user object in the database and store its id in `session[:guest_user_id]`.
方法是在数据库创建一个游客用户，将id存储在`session[:guest_user_id]`中。
 When (and if) the user registers or logs in, we delete the guest user and clear the session variable.
当（如果）用户注册或登录，就可以删除这个游客用户，清除会话变量。
 A helper function, `current_or_guest_user`, returns `guest_user` if the user is not logged in and `current_user` if the user is logged in.
辅助器函数`current_or_guest_user`返回`guest_user`，如果用户还没登录。如果用户登录，则是返回`current_user`。
 
```ruby
class ApplicationController < ActionController::Base

  protect_from_forgery

  # if user is logged in, return current_user, else return guest_user
  # 如果用户登录，返回current_user，否则返回guest_user。
  def current_or_guest_user
    if current_user
      if session[:guest_user_id] && session[:guest_user_id] != current_user.id
        logging_in
        guest_user(with_retry = false).try(:destroy)
        session[:guest_user_id] = nil
      end
      current_user
    else
      guest_user
    end
  end

  # find guest_user object associated with the current session,
  # creating one as needed
  # 找到guest_user对象相关的当前会话，必要时创建一个
  def guest_user(with_retry = true)
    # Cache the value the first time it's gotten.
    # 如果找到，在第一次找到时缓存该值。
    @cached_guest_user ||= User.find(session[:guest_user_id] ||= create_guest_user.id)

  rescue ActiveRecord::RecordNotFound 
  # if session[:guest_user_id] invalid 
  # 如果游客会话无效
     session[:guest_user_id] = nil
     guest_user if with_retry
  end

  private

  # called (once) when the user logs in, insert any code your application needs
  # to hand off from guest_user to current_user.
  # 只在用户登录时调用一次，插入处理从游客到转为当前用户的代码
  def logging_in
    # For example:
    # guest_comments = guest_user.comments.all
    # guest_comments.each do |comment|
      # comment.user_id = current_user.id
      # comment.save!
    # end
  end

  def create_guest_user
    u = User.create(:name => "guest", :email => "guest_#{Time.now.to_i}#{rand(100)}@example.com")
    u.save!(:validate => false)
    session[:guest_user_id] = u.id
    u
  end

end
```

Finally in order to fix the problem with ajax requests you have to turn off protect_from_forgery for the controller action with the ajax request:
最后为了修复ajax请求的问题，只能用ajax请求为控制器关闭protect_from_forgery。

```ruby
skip_before_filter :verify_authenticity_token, :only => [:name_of_your_action] 
```

Another option is to remove `protect_from_forgery` from `application_controller.rb` and put in each of your controllers and use :except on the ajax ones:
另一个选择是移除从应用控制器中`protect_from_forgery`，并将:except放在每个使用ajax的控制器中。

```ruby   
protect_from_forgery :except => :receive_guest
```

Last but not least, don't forget to add `helper_method :current_or_guest_user` to the controller to make the method accessible in views.
最后也是重要的，不要忘了将`helper_method :current_or_guest_user`添加到控制器中，要不在视图中就用不了该方法了。

### Authentication (this may interfere with the current_user helper)  认证（这可能涉及到current_user辅助器）

If you wish to continue using `before_filter :authenticate_user!` and have it apply to the guest user, you will need to add a new [Warden strategy](https://github.com/hassox/warden/wiki/Strategies).
如果要继续使用`before_filter :authenticate_user!`，并应用到游客用户，需要添加一个新的“警告策略”。
 For example: In initializers/some_initializer.rb
比如： 在 initializers/some_initializer.rb

```
Warden::Strategies.add(:guest_user) do
  def valid?
    session[:guest_user_id].present?
  end

  def authenticate!
    u = User.where(id: session[:guest_user_id]).first
    success!(u) if u.present?
  end
end
```

And in initializers/devise.rb
在initializers/devise.rb：

```
Devise.setup do |config|
  # ...
  config.warden do |manager|
    manager.default_strategies(scope: :user).unshift :guest_user
  end
end
```


    
