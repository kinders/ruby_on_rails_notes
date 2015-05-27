Devise 2.0 is actually a small release aimed mainly to clean up Devise source code. With time, Devise added new behaviors but, since we had to maintain backwards compatibility, we could not deprecate the old behaviors. Devise 2.0 finally deprecates such old behaviors.

The difficulty to migrate to Devise 2.0 will depend on when you started developing your application. If you started a long time ago, you will probably need to add or remove columns from your database schema. If recently, you will just need to rename a few configuration options.

**Before upgrading to Devise 2.0, be sure that your application is fine running on Devise 1.5.x and that you are running on at least Rails 3.1**

Devise 2.0 provides two main features:

* Support for e-mail reconfirmation when it changes. You can benefit from this change by setting `Devise.reconfirmable` to true and adding an `unconfirmed_email` column to your Devise models;
* Deprecation of Devise's migration methods. **All applications migrating to Devise 2.0 will have to upgrade their original Devise migrations. [This page describes exactly how to do it](https://github.com/plataformatec/devise/wiki/How-To:-Upgrade-to-Devise-2.0-migration-schema-style);**

Besides, a few configuration options were renamed or removed:

* `Devise.remember_across_browsers` is no longer effective. You can simply remove it;
* `Devise.stateless_token` was removed. If you want to have stateless tokens, simply do `config.skip_session_storage << :token_auth` in your initializer;

The locale information also changed, specifically:

```yaml
en:
  devise:
    registrations:
      inactive_signed_up: 'You have signed up successfully. However, we could not sign you in because your account is %{reason}.'
      reasons:
        inactive: 'inactive'
        unconfirmed: 'unconfirmed'
        locked: 'locked'
```

Should now be:

```yaml
en:
  devise:
    registrations:
      signed_up_but_unconfirmed: 'A message with a confirmation link has been sent to your email address. Please open the link to activate your account.'
      signed_up_but_inactive: 'You have signed up successfully. However, we could not sign you in because your account is not yet activated.'
      signed_up_but_locked: 'You have signed up successfully. However, we could not sign you in because your account is locked.'
```

If you also used Devise's `render_with_scope` method, you may notice that we removed it on 2.0 versions. If you used this method on your controllers (and have properly added your `scoped_path`) you should now only do a `render 'template'`, because Devise overrides Rails's `_prefixes` method and thus the correct path is added to the Rails lookup. 

Finally, in case you started using Devise in your app some time ago, your schema may be out of date and you need to update it:

* Devise now always uses the password salt as basis for the remember token. You can remove the `remember_token` column from your models and set `Devise.use_salt_as_remember_token` to true;
* Devise now requires you to have a `reset_password_sent_at` column. Please add it to your Devise models and set `Devise.reset_password_within` to an interval (like 6 hours);

Luckily, those are all changes you need to do to have your app successfully running on Devise 2.0. Welcome aboard!