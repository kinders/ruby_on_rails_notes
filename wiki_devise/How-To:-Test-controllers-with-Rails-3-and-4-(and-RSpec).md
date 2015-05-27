# vim: set ft=markdown :#
# Rails 3和Rails 4 的功能测试
First, be sure to [speed up your tests](https://github.com/plataformatec/devise/wiki/Speed-up-your-unit-tests)!
首先，确保加速测试！

## Controller tests (Test::Unit)  控制器测试

To sign up as admin for a given test case, just do:
对给定测试情景作为管理员登录，只需要：

```ruby
class SomeControllerTest < ActionController::TestCase
  include Devise::TestHelpers

  def setup
    @request.env["devise.mapping"] = Devise.mappings[:admin]
    sign_in FactoryGirl.create(:admin)
  end
end
```

Note: If you are using the confirmable module, you should set a `confirmed_at` date inside the Factory or call
`confirm!` before `sign_in`.
注意：如果使用确认模块，应该在Factory里面设置一个`confirmed_at`日期，或者在登录之前调用`confirm!`。

## Controller specs  控制器规格

Controller specs won't work out of the box if you're using any of devise's utility methods.
如果使用devise的utility方法，控制器规格不会在程序之外运行。

As of rspec-rails-2.0.0 and devise-1.1, the best way to put devise in your specs is simply to add the following into spec_helper:
到rspec-rails-2.0.0和devise-1.1，在规格钟放入devise的最好方法是，将下面的代码放到spec_helper中：

```ruby
require 'devise'

RSpec.configure do |config|
  config.include Devise::TestHelpers, :type => :controller
end
```

I also like to write *controller_macros.rb* file inside *spec/support* which contains the following:
也喜欢在spec/support中写controller_macros.rb文件，里面包含下面代码：

```ruby
module ControllerMacros
  def login_admin
    before(:each) do
      @request.env["devise.mapping"] = Devise.mappings[:admin]
      sign_in FactoryGirl.create(:admin) # Using factory girl as an example
    end
  end

  def login_user
    before(:each) do
      @request.env["devise.mapping"] = Devise.mappings[:user]
      user = FactoryGirl.create(:user)
      user.confirm! # or set a confirmed_at inside the factory. Only necessary if you are using the "confirmable" module
      sign_in user
    end
  end
end
```

Note: If your admin factory is nested on your user factory, you'll need to call `sign_in` like this:
注意：如果管理员因子嵌套在user因子之中，需要像下面这样调用`sign_in`：

```ruby
  def login_admin
    before(:each) do
      @request.env["devise.mapping"] = Devise.mappings[:admin]
      admin = FactoryGirl.create(:admin)
      sign_in :user, admin # sign_in(scope, resource)
    end
  end
```

Then in `spec/spec_helper.rb` or `spec/support/devise.rb`:
然后在`spec/spec_helper.rb`或`spec/support/devise.rb`中：

```ruby
RSpec.configure do |config|
  config.include Devise::TestHelpers, :type => :controller
  config.extend ControllerMacros, :type => :controller
end
```

So now in my controller specs I can now do:
所以在specs控制器中，可以这样做：

```ruby
describe MyController do
  login_admin

  it "should have a current_user" do
    # note the fact that I removed the "validate_session" parameter if this was a scaffold-generated controller
    subject.current_user.should_not be_nil
  end

  it "should get index" do
    # Note, rails 3.x scaffolding may add lines like get :index, {}, valid_session
    # the valid_session overrides the devise login. Remove the valid_session from your specs
    get 'index'
    response.should be_success
  end
end
```

## Mappings  映射

Every time you want to unit test a devise controller, you need to tell Devise which mapping to use.
每次想要对Devise控制器进行单元测试，需要告诉Devise使用哪个映射。
We need that because ActionController::TestCase and spec/controllers bypass the router and it is the router that tells Devise which resource is currently being accessed, you can do that with:
这是因为`ActionController::TestCase`和spec/controllers会绕过路由，路由会告诉Devise正在访问哪个资源。
可以这样：

```ruby
@request.env["devise.mapping"] = Devise.mappings[:admin]
```

## Authenticated routes in Rails 3

If you choose to authenticate in routes.rb, you lose the ability to test your routes via assert_routing (which combines assert_recognizes and assert_generates, so you lose them also).
如果选择在路由中选择认证，会丧失通过assert_routing测试路由的能力；也不能将assert_routing和assert_recognizes、assert_generates连用。
It's a limitation in Rails: Rack runs first and checks your routing information but since functional/controller tests run at the controller level, you cannot provide authentication information to Rack which means `request.env['warden']` is nil and Devise generates one of the following errors:
在Rails中这是一个限制：Rack首先运行，检查路由信息，但因为功能/控制器测试在控制器级别上运行，你不能提供认证信息给Rack，这就意味着`request.env['warden']`为nil，Devise因此会生成以下几个中的一个错误：


```text
NoMethodError: undefined method 'authenticate!' for nil:NilClass
NoMethodError: undefined method 'authenticate?' for nil:NilClass
```

The solution is to test authenticated routes in the *controller* tests.
To do this, stub out your authentication methods for the controller test, as described here: 

[How-To: Stub authentication in controller specs](https://github.com/plataformatec/devise/wiki/How-To:-Stub-authentication-in-controller-specs)

## Rails 3 RSpec scaffolds

If you're using the default Rspec scaffold generator, the generated controller specs pass along session parameters:
如果使用默认的Rspec支架生成器，生成的控制器规格传入会话参数：

```get :index, {}, valid_session```

These are overwriting the session variables that Devise's helpers set to sign in with Warden.
这会改写会话变量，该变量是Devise的辅助器设置来使用Warden登录的。
The simplest solution is to remove them:
最简单的解决方案是移除它们：


```get :index, {}```

Alternatively, you could set the Warden session information in them manually, instead of using Devise's helpers.
可选的，可以手动在里面设置Warden会话信息，而不是使用Devise的辅助器：

credit: [Ian Terrell's answer on stackoverflow](http://stackoverflow.com/a/9658682/151899)

