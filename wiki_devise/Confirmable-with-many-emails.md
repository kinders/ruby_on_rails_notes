Suppose you have a pattern like
User has_many :emails
(where one of the email acts as a :primary email)

You can use confirmable option in the following way.
In your User model use

    devise :confirmable, ...

    has_many :emails
    delegate :confirmation_sent_at, :confirmed_at, :confirmation_token, to: :primary_email

    def primary_email
      emails.primary || (emails.first if new_record?)
    end

This means your :confirmation_sent_at, :confirmed_at, :confirmation_token, :primary columns will reside in Email model