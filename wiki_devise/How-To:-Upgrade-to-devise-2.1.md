We're supposing that you are currently using Devise 2.0, if not, before doing any of the tips below, [upgrade your app to Devise 2.0](https://github.com/plataformatec/devise/wiki/How-To:-Upgrade-to-Devise-2.0)

### Deprecations added on 2.0 are now removed

If you run your application with Devise 2.0 and have removed all the deprecation warnings, you're good to go with Devise 2.1.

### Devise.use_salt_as_remember_token is removed

In Devise 2.0, the only acceptable value for this option was true, since we no longer maintained a remember_token. Since it always had to be true, the option was removed all together (we had planned a two steps deprecation since 2.0 release).

### Encryptable now is in a gem

If you're using encryptable on your models, you will now have to include `devise-encryptable` on your gemfile.

### Sign out through JSON or XML format now returns a contentless header

Now the `sign_out` through JSON or XML returns a contentless response instead of a response with an empty body.

### devise/links partial changed.

The `views/devise/_links.html.erb` has been moved back to `views/devise/shared/_links.html.erb` so if you
updated customized views in 2.0 you will need to update the path.