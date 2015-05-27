Sometimes you want to add custom validation to the user before logging them in. In this case I needed to implement an account_active boolean (true or false) check. So if it's true it will allow the user to login and create a session, if false it will display the "account not active" error.

Overwrite the `active_for_authentication?` method in your model (User) and add your validation logic. You want to return super && (true or false)

```ruby
      def active_for_authentication?
        # Uncomment the below debug statement to view the properties of the returned self model values.
        # logger.debug self.to_yaml
        	
        super && account_active?
      end
```

In this case it checks the boolean value to account_active and it return it's value to the sign in method that called it.

The original `active_for_authentication?` method can be found in `devise/lib/devise/models/authenticatable.rb`.

Note: `active_for_authentication?` is called by Devise after authenticating user **and in each request that follows**. This means whatever code you add to the override of `active_for_authentication?` will be run for every request during the user's session.