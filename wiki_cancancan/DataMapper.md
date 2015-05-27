CanCan 1.5 adds support for [[DataMapper|http://datamapper.org/]]. All you have to do is mention `dm-core` before `cancan` in your Gemfile so it is required first.

```ruby
gem "dm-core"
gem "cancan"
```

That is it, you can now call `accessible_by` on any DataMapper model (which is done automatically in the `index` action). You can also use the query syntax that DataMapper provides when defining the abilities.

```ruby
# in Ability
can :read, Article, :priority.lt => 5
cannot :manage, Article, :priority.gte => 5
```

This is all done through a [[Model Adapter]]. See that page for more information and how you can add your own.