# How To: Add timeout in value dynamically #

## This feature was added in version 1.5.2 and is not available in older versions of Devise ##

To dynamically set the timeout for each user, you can define a method in the user model called timeout_in that
returns the timeout value.

    class User < ActiveRecord::Base
      devise (...), :timeoutable

      def timeout_in
        if self.admin? 
          1.year
        else
          2.days
        end
      end
    end

`timeout_in` should return a integer with the number of
seconds (the `timedout?` method that devise uses, call the `ago` method on what timeout
returns). But of course, you can use Rails `10.seconds` or `1.hour`
to improve readability.
