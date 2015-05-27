We have created paranoid mode to avoid enumerating users. First, if you don't know anything about user enumeration, take a look at this reference:

* [OWASP testing for user enumeration](http://www.owasp.org/index.php/Testing_for_user_enumeration_(OWASP-AT-002))

If you use Paranoid-mode on Devise, you're protected against user enumeration on confirmable, recoverable and unlockable modules, but not on registerable. One of the validations on creating a new user is for it to have an unique e-mail or login. So, we can't add a response that s to the register controller because the user will not know if his account was created or not. 

There are two solutions that are very common in the internet, that should stop robots doing the enumeration:

* Add a captcha;
* Add a rule that blocks create requests for a few minutes after creating a small number of users. E.g. blocking an IP for five minutes after creating five users.

Of course, it only stops robots doing a lot of requests. There is no way to stop anybody doing an enumeration by hand.

In order to use the parameter, you have to turn on the paranoid mode on your devise.rb, like this:

``` ruby
Devise.setup do |config|
  config.paranoid = true
end
```

