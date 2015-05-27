## Actions
* `:read => [:index, :show]`
* `:create => [:new]`
* `:update => [:edit]`
* `:destroy => [:delete]`
* `:access` is an alias for all actions

You can add additional action aliases by calling `alias_action` from the `Ability` class `initialize` method. For instance, to add the `search` and `query` actions to the `read` alias: `alias_action(:search, :query, :to => :read)`

## Subject
* `:all` is an alias for all subjects

You can add additional subject aliases by calling `alias_subject` from the `Ability` class `initialize` method. For instance, to add a group of API controllers as a single subject: `alias_subject(:'api/users', :'api/locations', :to => :api)`

## Notes
* `:all` and `:access` seem to be 'special' in that they are not listed when you ask an instance of the `Ability` class for its aliases via `ability.aliases`

* You can check the currently defined aliases in rails console by initializing the ability class with something like `a = Ability.new(User.first)` and then running `a.aliases`.

* When defining abilities or setting aliases for namespaced controllers, you must use a quoted symbol such as `:'api/users'`. A simple string (`'api/users'`) will **not** work.
