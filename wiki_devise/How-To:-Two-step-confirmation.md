There are at least two approaches in the wiki for two-step confirmation:
维基里只有有两种方法实现两步验证：

[How To: Email only sign up](https://github.com/plataformatec/devise/wiki/How-To:-Email-only-sign-up) -- this is the method first described by Claudio Marai.
It is a simple method that allows user account creation with only an email address.
The new user is sent a confirmation email with a link.
Clicking on that link loads a webpage where they must submit a valid password before their account is marked as confirmed.
《电邮注册》——这种方法最早由Claudio Marai提出。
这种简单的方法允许用户账户通过一个电邮地址创建。
单击该链接将导入一个网页，用户必须提交一个可用的密码，账户才被确认。
 
[How To: Override confirmations so users can pick their own passwords as part of confirmation activation](https://github.com/plataformatec/devise/wiki/How-To:-Override-confirmations-so-users-can-pick-their-own-passwords-as-part-of-confirmation-activation) -- is an older, more complex method that accomplishes the same thing.
《改写确认，让用户在确认激活时选择一个密码》是一个更老、更复杂的方法，效果一样。
