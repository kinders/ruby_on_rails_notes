Devise 2.0 no longer includes helpers for your migrations (even for your old migrations creating tables used in versions prior to 2.0). Instead, we simply list the database fields explicitly. This page shows the before and after of your original Devise migration(s) so you can easily compare and update your code.

Meaning: You will need to change your *original* Devise migration(s) to not use the Before helpers, but instead use the fields listed in After, here.  Be sure to check all of your migrations, not just your initial migration creating your User table (for instance), for those migrations where you might have added additional Devise features and previously used the helpers.

## Active Record

Below follows the generated code before and after Devise 2.0. You can use it as guideline but also consult your schema.rb for extra help.

### Before

<pre lang="ruby">
create_table(TABLE_NAME) do |t|
  t.database_authenticatable :null => false
  t.recoverable
  t.rememberable
  t.trackable

  # t.encryptable
  # t.confirmable
  # t.lockable :lock_strategy => :failed_attempts, :unlock_strategy => :both
  # t.token_authenticatable
end
</pre>

### After

<pre lang="ruby">
create_table(TABLE_NAME) do |t|
  ## Database authenticatable
  t.string :email,              :null => false, :default => ""
  t.string :encrypted_password, :null => false, :default => ""

  ## Recoverable
  t.string   :reset_password_token
  t.datetime :reset_password_sent_at

  ## Rememberable
  t.datetime :remember_created_at

  ## Trackable
  t.integer  :sign_in_count, :default => 0
  t.datetime :current_sign_in_at
  t.datetime :last_sign_in_at
  t.string   :current_sign_in_ip
  t.string   :last_sign_in_ip

  ## Encryptable
  # t.string :password_salt

  ## Confirmable
  # t.string   :confirmation_token
  # t.datetime :confirmed_at
  # t.datetime :confirmation_sent_at
  # t.string   :unconfirmed_email # Only if using reconfirmable

  ## Lockable
  # t.integer  :failed_attempts, :default => 0 # Only if lock strategy is :failed_attempts
  # t.string   :unlock_token # Only if unlock strategy is :email or :both
  # t.datetime :locked_at

  # Token authenticatable
  # t.string :authentication_token

  ## Invitable
  # t.string :invitation_token

  t.timestamps
end
</pre>

## Mongoid

Before Devise 2.0, Devise automatically configured your MongoDB. This is no longer true. In order to upgrade, you need to set `Devise.apply_schema` to false and add the following to your model according to the strategies you are using:

<pre lang="ruby">
  ## Database authenticatable
  field :email,              :type => String, :null => false
  field :encrypted_password, :type => String, :null => false

  ## Recoverable
  field :reset_password_token,   :type => String
  field :reset_password_sent_at, :type => Time

  ## Rememberable
  field :remember_created_at, :type => Time

  ## Trackable
  field :sign_in_count,      :type => Integer
  field :current_sign_in_at, :type => Time
  field :last_sign_in_at,    :type => Time
  field :current_sign_in_ip, :type => String
  field :last_sign_in_ip,    :type => String

  ## Encryptable
  # field :password_salt, :type => String

  ## Confirmable
  # field :confirmation_token,   :type => String
  # field :confirmed_at,         :type => Time
  # field :confirmation_sent_at, :type => Time
  # field :unconfirmed_email,    :type => String # Only if using reconfirmable

  ## Lockable
  # field :failed_attempts, :type => Integer # Only if lock strategy is :failed_attempts
  # field :unlock_token,    :type => String # Only if unlock strategy is :email or :both
  # field :locked_at,       :type => Time

  # Token authenticatable
  # field :authentication_token, :type => String

  ## Invitable
  # field :invitation_token, :type => String
</pre>