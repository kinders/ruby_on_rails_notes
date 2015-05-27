To create your own serializer, just change this:
``` ruby
  config.warden do |manager|
    manager.serialize_into_session(:your_scope) do |user|
      user.id
    end

    manager.serialize_from_session(:your_scope) do |key|
      User.find(key)
    end
  end
```
in your __/config/initializers/devise.rb__ file.

For example, if you have more than one class in one scope that needs to be serialized and deserialized without getting the wrong class, you can do something like this:
``` ruby
  config.warden do |manager|
    manager.serialize_into_session(:user) do |user|
      [user.class, user.id]
    end

    manager.serialize_from_session(:user) do |keys|
      klass, id = keys
      klass.constantize.find(id)
    end
  end
```
In this specific case, we were using our own warden strategies without any **authenticatable** strategy from devise.