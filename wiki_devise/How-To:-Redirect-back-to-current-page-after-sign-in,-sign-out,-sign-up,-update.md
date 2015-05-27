# 在注册、登录、登出、更新之后重定向到当前页面
## Required Reading 必读

This wiki page is fairly out-of-date, just read the code: [`lib/devise/controllers/store_location.rb`][1]
这个维基页面已经相当过时了，请阅读lib中的源代码。

## Code 代码
In ```ApplicationController``` write following:

```ruby
after_filter :store_location

def store_location
  # store last url - this is needed for post-login redirect to whatever the user last visited.
  return unless request.get? 
  if (request.path != "/users/sign_in" &&
      request.path != "/users/sign_up" &&
      request.path != "/users/password/new" &&
      request.path != "/users/password/edit" &&
      request.path != "/users/confirmation" &&
      request.path != "/users/sign_out" &&
      !request.xhr?) # don't store ajax calls
    session[:previous_url] = request.fullpath 
  end
end

def after_sign_in_path_for(resource)
  session[:previous_url] || root_path
end
```
Note that if you happen to have ```before_filter :authenticate_user!``` in your ```ApplicationController``` you will need to use ```before_filter :store_location``` rather than ```after_filter :store_location``` and make sure to place it ahead of ```before_filter :authenticate_user!```. This is due to the likelihood that if a user fails to be authenticated they will by default be redirected to ```/users/sign_in``` and thus not appropriately set the ```session[:previous_url]``` value.

Also, note that some types of requests you receive you may not want to store as a previous URL, for example JS or JSON requests. If you stored such a URL, the user went away, and later returned and this was still stored in their session, they will be redirected to a URL that they probably did not intend. Additionally, it may make sense to ensure the previously requested URL is recent and not from an abandoned session long ago. This could look like:
```ruby
if request.format == "text/html" || request.content_type == "text/html"
  session[:previous_url] = request.fullpath
  session[:last_request_time] = Time.now.utc.to_i
end
```

Then, in your session controller's after_sign_in_path_for(resource) method, you can now check to make sure the request is recent enough that you want to redirect the user.

Alternatively, you can simply modify the regular expression comparison (```/\/users/``` in the above example) to only match the specific urls that you wish to skip, such as ```/\/users\/sign_in/```.

You may also want to add
```ruby
def after_update_path_for(resource)
  session[:previous_url] || root_path
end
```
or
```ruby
def after_sign_out_path_for(resource)
  session[:previous_url] || root_path
end
```
Be aware, however, that there may be resources available to a signed-in user that would not be available to the anonymous user, so use the ```after_sign_out_path_for``` with caution.

## Explanation  解释
We only want to keep track of a previous url if it is not a /users url.  This way, it will redirect to the last url that is not involved in the user sign in, sign up, or update process.

***

### Problems when used along with confirmable module
There is a change at least in Devise 3.03 and Rails 4. If you user should confirm the mail as well along with this system you will get problems where he will log in on confirmation but he will redirected to the resend confirmation instructions page. So,modifying the code to be like this will solve the problem

    def store_location
     if (!request.fullpath.match("/users") &&
      !request.xhr?) # don't store ajax calls
      session[:previous_url] = request.fullpath
     end
    end

****
###A Simpler Solution?
Note that devise actually sets the last request for you when a user fails to authenticate. This occurs here and here in store_location! and store_location_for respectively as part of the failure scenario.

This means that writing your own solution for storing the session is redundant and not necessary for every single request that hits your application. The below code examples should suffice:

```
class ApplicationController < ActionController::Base


private

# If your model is called User
def after_sign_in_path_for(resource)
  session["user_return_to"] || root_path
end

# Or if you need to blacklist for some reason
def after_sign_in_path_for(resource)
  blacklist = [new_user_session_path, new_user_registration_path] # etc...
  last_url = session["user_return_to"]
  if blacklist.include?(last_url)
    root_path
  else
    last_url
  end
end

end
```
Note that blacklisting might not be necessary for your application since the session["user_return_to"] value is set only on an authentication failure.

In order for this to work, you must place make use of devise's `authenticate_user!` method. You can either place this in a before_filter on your application controller or in each individual controller. 

If you have named your resource something else other than `User` then change the resource method names accordingly.

[1]: https://github.com/plataformatec/devise/blob/master/lib/devise/controllers/store_location.rb
