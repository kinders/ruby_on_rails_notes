# How to use Devise Inside a Mountable Engine #

Installing Devise in a mountable engine is nearly identical to installing Devise in an application. However, there are a few additional configuration options that need to be adjusted.

## Warning ##

It's not possible to mount Devise multiple times ([related issue w/ a possible workaround](https://github.com/plataformatec/devise/issues/2827)). For instance, if you have a main Rails app called `A` which mounts an engine `B`, you can't have Devise mounted for both `A` and `B`. `A`'s `config/initializer/devise.rb` will be overwritten by `B`, which breaks options like `config.parent_controller` and `config.omniauth_path_prefix`.

## Installation ##
Installing Devise in an engine follows the same steps as the [general installation instructions](https://github.com/plataformatec/devise#getting-started), but we'll be running the commands from our engine's directory and benefiting from the namespaced `rails` helper there.

Add Devise to the engine's dependencies in your gemspec:
```ruby
Gem::Specification.new do |s|
  s.add_dependency "devise"
end
```

Generate the config files:

```bash
rails generate devise:install
```

And generate a model if you need to:

```bash
rails generate devise MODEL
```

## Configuration ##

You'll need to direct Devise to use your engine's router. To do this, set `Devise.router_name` in `config/initializers/devise.rb`. It should be a symbol containing the name of the mountable engine's named-route set. For example, if your engine is namespaced as `MyEngine`:
```ruby
Devise.setup do |config|
  config.router_name = :my_engine
end
```

Your user class is probably namespaced, so you'll need to pass that to the Devise helper in `routes.rb`:
```ruby
MyEngine::Engine.routes.draw do
  devise_for :users, class_name: "MyEngine::User"
end
```

If your engine uses `isolate_namespace`, Devise will assume that all of its controllers reside in your engine rather than Devise. To correct this, add `:module => :devise` to the routes helper:
```ruby
MyEngine::Engine.routes.draw do
  devise_for :users, class_name: "MyEngine::User", module: :devise
end
```

You probably want Devise's controllers to inherit from your engine's controller and not the main controller. Set this in `config/initializers/devise.rb`:
```ruby
Devise.setup do |config|
  config.parent_controller = 'MyEngine::ApplicationController'
end
```

Finally, you'll need to require Devise in your engine. Add the following line to `lib/my_engine.rb`:

```ruby
require 'devise'
```
You can also set the layout for specific Devise controllers using a callback in `lib/my_engine.rb`:

```ruby
module MyEngine
  class Engine < ::Rails::Engine
    config.to_prepare do
      Devise::SessionsController.layout "layout_for_sessions_controller"
    end
  end
end
```

NOTE: If you need to override the standard devise views (i.e. app/views/devise/sessions/new) you will want to require 'devise' before you require your engine. This way the view order will get configured appropriately and you can manage the overrides within your engine rather than the main app.

## Path helpers ##
Any code that gets run from the context of a Devise controller (controllers, helpers, and views) may require using "fully-qualified" path helpers, e.g. `my_engine.root_path`. The path helpers defined by Devise don't need this. I'm not sure if this is a bug.

## Example ##

http://github.com/BrucePerens/perens-instant-user/
is an example Rails Engine that adds Devise to the application while keeping most of the complexity of using Devise in the engine rather than your application.

It contains pre-defined mountable routes for Devise, and a pre-defined User model. Its installation steps are easier than those of stand-alone Devise. Unlike stand-alone Devise, you aren't advised to become a Ruby Wizard before installing it :-)

## Omniauth ##

There is an issue with using Devise's Omniauth integration in an embedded engine. See [this issue](https://github.com/plataformatec/devise/issues/2692) for workarounds.