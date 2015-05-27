When I do the following in my rspec tests:
sign_in User.make!(:admin)

I get the following error 'Email has already been taken'.

Machinist tries to cache the user and it just doesn't work as it starts complaining about the 'Email has already been taken'.

So to fix this follow:
https://github.com/notahat/machinist/wiki/Object-Caching

For rspec I just required the following in my environments/test.rb file:

    Machinist.configure do |config|
      config.cache_objects = false
    end

You may need to run:
`rake db:test:prepare`