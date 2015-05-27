Often (like, when creating users as part of another controllers create method) you want to sign in the user from a controller. 
This is how it is accomplished:
我们经常需要，比如创建用户是其他控制器的create方法的一部分，从一个控制器中登录用户。
下面是代码：

`sign_in(:user, User.find(params[:id]))`

Of course you could also just pass in a predefined User object.
当然也可以传入一个定义好的User对象
