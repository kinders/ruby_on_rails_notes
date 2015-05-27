# 为所有Devise邮箱设置主机和端口
If you find yourself needing to set a :host and :port for your mailer url's (e.g. when sending a reset password link) [follow the instructions for overriding the devise mailer](https://github.com/plataformatec/devise/wiki/How-To:-Use-custom-mailer) and then follow these steps:
如果发现自己需要为邮箱地址设置主机和端口（比如当发送一个重设密码的连接），《跟随指令，改写devise邮箱》，然后跟着下面的步骤：

Create a file in app/concerns called default_url_options.rb.
在app/concerns中创建一个叫default_url_options.rb的文件。
Depending on your version of Rails you might need to add a setting to your environment that loads files from this folder automatically.
取决于Rails的版本，可能需要添加一个环境设置，从这个目录自动加载文件。

In this file put something like the following:
在这个文件放入类似下面的东西：

    module DefaultUrlOptions
      
      # Including this file sets the default url options. This is useful for mailers or background jobs
      # 这对邮箱后台作业十分有用。
      
      def default_url_options
        {
          :host => host,
          :port => port
        }
      end
      
    private
  
      def host
        # Your logic for figuring out what the hostname should be
      end
      
      def port
        # Your logic for figuring out what the port should be
      end
      
    end

Next, just include this module in your devise mailer:
接下来，在devise邮箱模块中包含它：

    class UserMailer < Devise::Mailer   
      include DefaultUrlOptions
    
    end

It's worthwhile noting that you'll need to be able to figure out the host and port in the above example without having access to any request object as mailers and background jobs run outside the context of the request.
上面这个例子很不值得，需要能够描述主机和端口，无需访问任何请求对象作为邮箱，后台作业在请求的环境外运行。<kinder:note> 让我这么翻译？？晕了。
