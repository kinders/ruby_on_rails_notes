### Backwards incompatible changes (from CHANGELOG.md)

https://github.com/plataformatec/devise/blob/master/CHANGELOG.md

* Do not store confirmation, unlock and reset password tokens directly in the database. This means tokens previously stored in the database are no longer valid. You can reenable this temporarily by setting `config.allow_insecure_token_lookup = true` in your configuration file. It is recommended to keep this configuration set to true just temporarily in your production servers only to aid migration
* The Devise mailer and its views were changed to explicitly receive a token argument as `@token`. You will need to update your mailers and re-copy the views to your application with `rails g devise:views`
* Sanitization of parameters should be done by calling `devise_parameter_sanitizer.sanitize(:action)` instead of `devise_parameter_sanitizer.for(:action)`

### Further Reading

* http://joanswork.com/devise-3-1-update/