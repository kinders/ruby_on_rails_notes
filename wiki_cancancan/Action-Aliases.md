You will usually be working with four actions when [[defining|Defining Abilities]] and [[checking|Checking Abilities]] permissions: `:read`, `:create`, `:update`, `:destroy`.
在定义能力和检查能力时，经常会用到四种动作：读取，创建，更新，删除
These aren't the same as the 7 RESTful actions in Rails.
这和Rails里的表转风格的七种动作不同。
CanCan automatically adds some convenient aliases for mapping the controller actions.
CanCan自动增加了一些方便的别名映射到控制器的动作中。
 
```ruby
alias_action :index, :show, :to => :read
alias_action :new, :to => :create
alias_action :edit, :to => :update
```

Notice the `edit` action is aliased to `update`. 
注意编辑动作被别名为更新。
This means if the user is able to update a record he also has permission to edit it. 
这意味着如果用户能够更新记录，那他就能进行编辑。

### 自定义别名
You can define your own aliases in the `Ability` class.
可以在`Ability`类中定义自己的别名。

```ruby
class Ability
  include CanCan::Ability
  def initialize(user)
    alias_action :update, :destroy, :to => :modify
    can :modify, Comment
  end
end

# in controller or view
can? :update, Comment # => true
```

You are not restricted to just the 7 RESTful actions, you can use any action name.
可以使用其他的动作名，不要拘束于这7中表转风格的动作。
See [[Custom Actions]] for details.
详见参见《自定义动作》。

### 修改默认别名
Please note that if you are changing the default alias_actions, the original actions associated with the alias will NOT be removed.
注意如果改变了默认的动作别名，与这个别名关联的原始的动作被没有被删除。
 For example, following statement will not have any effect on the alias :read, which points to :show and :index: 
比如，下面的声明不能影响别名“:read”，它指向“:show”和“:index”。

```ruby
alias_action :show, :to => :read # this will have no effect on the alias :read!
```

If you want to change the default actions, you should use clear_aliased_actions method to remove ALL default aliases first.
如果想改变默认动作，应该先使用clear_aliased_actions方法来移除所有默认别名。
