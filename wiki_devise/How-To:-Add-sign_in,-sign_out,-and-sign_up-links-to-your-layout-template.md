First add sign_in/out links, so the appropriate one will show up depending on whether the user is _already_ signed in:

```erb
# views/devise/menu/_login_items.html.erb
<% if user_signed_in? %>
  <li>
  <%= link_to('Logout', destroy_user_session_path, :method => :delete) %>        
  </li>
<% else %>
  <li>
  <%= link_to('Login', new_user_session_path)  %>  
  </li>
<% end %>
```

The <code>:method => :delete</code> part is required if you use the default HTTP method.  To change it, you will need to tell Devise this:

```ruby
# config/initializers/devise.rb
  # The default HTTP method used to sign out a resource. Default is :delete.
  config.sign_out_via = :get
```

You can then omit <code>:method => :delete</code> on all your sign_out links.

Next come the sign_up links.  Again, these can be substituted with something else useful if the user is already signed in:

```erb
# views/devise/menu/_registration_items.html.erb
<% if user_signed_in? %>
  <li>
  <%= link_to('Edit registration', edit_user_registration_path) %>
  </li>
<% else %>
  <li>
  <%= link_to('Register', new_user_registration_path)  %>
  </li>
<% end %>
```

Then use these templates in your <code>layouts/application.html.erb</code>, like this:

```erb
# layouts/application.html.erb
<ul class="hmenu">
  <%= render 'devise/menu/registration_items' %>
  <%= render 'devise/menu/login_items' %>
</ul>
<%= yield %>
```

Add some menu styling to the CSS (here for a horizontal menu):

```css
ul.hmenu {
  list-style: none;	
  margin: 0 0 2em;
  padding: 0;
}

ul.hmenu li {
  display: inline;  
}
```
