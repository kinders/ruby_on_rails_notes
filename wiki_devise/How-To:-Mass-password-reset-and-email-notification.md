This code help you to assign a random password for each user and send a email notification to them:

**source:** https://gist.github.com/1160873

```ruby
# Assign an new random password for each user and send them a email notification
# path: lib/tasks/devise.rake

namespace :devise do
  desc 'Mass password reset and send email instructions'
  task :mass_password_reset => :environment do
    model = ENV['MODEL'] || 'User'
    begin
      model_mailer = "#{model}Mailer".constantize
      model = model.constantize

      model.find_each(:conditions => 'id > 70720') do |record|
        # Assign a random password
        random_password = User.send(:generate_token, 'encrypted_password').slice(0, 8)
        record.password = random_password
        record.save
        # Send change notification (Ensure you have created #{model}Mailer e.g. UserMailer)
        model_mailer.password_reset(record, random_password).deliver  
      end
    rescue Exception => e
      puts "Error: #{e.message}"
    end
  end
end
```

And create the mailer:

```ruby
# Send password reset notification
# path: app/mailers/user_mailer.rb
class UserMailer < ActionMailer::Base
  default :from => "no-reply@example.com"

  def password_reset(user, password)
    @user = user
    @password = password
    mail(:to => user.email,
         :subject => 'Password Reset Notification')
  end
end
```