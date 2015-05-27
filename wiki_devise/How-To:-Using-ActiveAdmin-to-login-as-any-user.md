As a way to provide support to end users, the ability to login to their rails app as a specific user can be useful.
If you're using ActiveAdmin and Devise, adding this feature takes less than 10 minutes.

The feature consisted of two changes, customising the ActiveAdmin User show page and adding a custom member action that logs in the user.

_Code Snippet 1-a: show page_

``` ruby
show do |user|
    attributes_table do
      #We want to keep the existing columns
      User.column_names.each do |column|
        row column
      end
      #This is where we add a new column
      row :login_as do
        link_to "#{user.name}", login_as_admin_user_path(user), :target => '_blank'
      end
    end
  end
```
Next we'll add the login_as member action to ActiveAdmin ([http://www.activeadmin.info/docs/8-custom-actions.html](http://www.activeadmin.info/docs/8-custom-actions.html))

_Code Snippet 1-b: member action_

``` ruby
  # Allows admins to login as a user 
  member_action :login_as, :method => :get do
    user = User.find(params[:id])
    sign_in(user, bypass: true)
    redirect_to account_path 
  end
```

....And there you have it.
Now admin users can quickly log in as any user for quick trouble shooting, user testing, etc.

Taken from [http://www.metachunk.com/blog/using-activeadmin-login-any-user-0](http://www.metachunk.com/blog/using-activeadmin-login-any-user-0) on 27th July 2013.
