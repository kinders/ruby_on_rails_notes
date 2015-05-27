There are a few things that need to be done when upgrading from 2.1 to 2.2.

### Add new i18n key

Add the following key between ```devise.failure.locked``` and ```devise.failure.invalid``` in the config/locales/devise.en.yml.

    not_found_in_database: 'Invalid email or password.'

### Add additional parameter to any Mailers you have overridden

For example:

    def confirmation_instructions(record)

becomes

    def confirmation_instructions(record, opts={})
    

## Optional

### Add responding to JSON

Devise no longer responds to JSON by default.  To add this ability, add the following to your ```config/application.rb```.

    config.to_prepare do
       DeviseController.respond_to :html, :json
    end

### Keeping password minimum length 6

The default minimum password length was changed to 8. To set it back to 6, add the following to ```config/initializers/devise.rb```.

    config.password_length = 6..128
