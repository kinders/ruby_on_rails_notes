To disable users from destroying their accounts just redefine routes for registrations controller without route to destroy action, so for example:

``` ruby
devise_for :users, skip: :registrations
devise_scope :user do
  resource :registration,
    only: [:new, :create, :edit, :update],
    path: 'users',
    path_names: { new: 'sign_up' },
    controller: 'devise/registrations',
    as: :user_registration do
      get :cancel
    end
end
```

For more details see devise_registration method from lib/devise/rails/routes.rb.