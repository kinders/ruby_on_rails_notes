# Devise + Authority + rolify Tutorial

This Tutorial shows you how to setup a [Rails >=3.1](http://rubyonrails.org/) application with a strong and flexible authentication/authorization stack using [Devise](https://github.com/plataformatec/devise), [Authority](https://github.com/nathanl/authority) and [rolify](https://github.com/EppO/rolify) (3.0 and later).

The basic idea is:
* [Devise](http://github.com/plataformatec/devise) provides authentication: lets users sign up and sign in, so that you know who they are.
* [rolify](http://github.com/EppO/rolify) helps you assign roles to users and check which roles they have.
* [Authority](http://github.com/nathanl/authority) helps you use those roles, or any other logic you like (user points, info from a single sign on app, etc) to control who can do what.

# Installation

1. First, create a bare new rails app. If you already have an existing app with Devise and Authority set up and you just want to add rolify, just add rolify in your Gemfile, run `bundle install` and skip to step 6
  * `# rails new rolify_tutorial`
  * edit the `Gemfile` and add Devise, Authority and rolify gems:
```ruby
gem 'devise'
gem 'authority'
gem 'rolify'
```

2. run `bundle install` to install all required gems

3. Run Devise generator
  * `# rails generate devise:install`

4. Create the User model from Devise
  * `# rails generate devise User`

5. Create the ApplicationAuthorizer from Authority
  * `# rails generate authority:install`

6. Create the Role class from rolify
  * `# rails generate rolify Role User`

7. Run migrations
  * `# rake db:migrate`

# Configuration

1. Configure Devise according to your needs. Follow the [Devise README](https://github.com/plataformatec/devise/blob/master/README.md) for details.

2. Edit the ApplicationAuthorizer (created by authority), updating the `default` method to check for the admin role:

```ruby
  def self.default(adjective, user)
    user.has_role? :admin
  end
```

This represents that all actions require the admin role by default. You can then begin adding more specific logic.

```ruby
  # To update a specific resource instance, you must either own it or be an admin
  def updatable_by?(user)
    resource.author == user || user.has_role?(:admin)
  end
```

3. Use rolify's `resourcify` method in all models you want to put a role on. For example, if we have the Post model:

```ruby
class Post < ActiveRecord::Base
  resourcify
  belongs_to :author, class_name: 'User'
end
```
4. `include Authority::UserAbilities` in your User class (to add methods like `can_update?`) and `include Authority::Abilities` in your models (to add methods like `updatable_by?`), which will delegate to ApplicationAuthorizer unless you specify a different authorizer class.

```ruby
class User < ActiveRecord::Base
  include Authority::UserAbilities
  has_many :posts, foreign_key: :author_id
end

...

class Post < ActiveRecord::Base
  resourcify
  include Authority::Abilities
end
```

# Usage

1. Create some User instances using `rails console`

```ruby
> alice = User.new
> alice.email = "alice@example.com"
> alice.password = "test1234"
> alice.save

> bob = User.new
> bob.email = "bob@example.com"
> bob.password = "test1234"
> bob.save

> cathy = User.new
> cathy.email = "cathy@example.com"
> cathy.password = "test1234"
> cathy.save
```

2. Add the admin role to a user.

```ruby
> alice.add_role "admin"
```

3. Create a post for a user:

```ruby
> post = bob.posts.new
> post.title = 'Test Post'
> post.name = 'Test Post'
> post.content = 'Nothing to see here'
> post.save
```

3. Check who can update this post.

```ruby
> bob.can_update?(post) #=> true; he is the author
> alice.can_update?(post) #=> true; she is an admin
> cathy.can_update?(post) #=> false
```

# Configure Authority

Authority provides ways to add enforcement of user authorization to your controllers. You can also configure which adjectives (like `updatable_by?`) and which verbs (like `can_update?`) are available in your application. For more info, see the [Authority README](http://github.com/nathanl/authority).