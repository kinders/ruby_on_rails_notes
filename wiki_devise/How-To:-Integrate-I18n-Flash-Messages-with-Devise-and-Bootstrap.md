# 用devise和Bootstrap整合国际化闪存信息
This tutorial was made using Rails 4 and Devise 3 and Bootstrap 2.3
这个指南使用Rails4和Devise3、Bootstrap 2.3。

This tutorial is made to show you how to integrate devise flash messages with twitter bootstrap.
这个指南展示了怎么用Bootstrap整合devise闪存信息。
I wanted to show a way to make devise flash messages look the same as the rest of the site.
我想展示一个方法让devise闪存看起来和网站的其他部分一样。
This will handle normal flash messages and devise flash messages.
它会处理普通的闪存信息和devise闪存信息。

<h3>Flash Messages For the Site  网站的闪存信息</h3>

First we will make a rendered view to make the code concise.
首先做个呈现视图，让代码简明化。
Within **"app/views/layouts/application.html.erb"** I added `<%= render 'layouts/messages' %>`.
在app/views/layouts/application.html.erb里添加`<%= render 'layouts/messages' %>`

My file looks like:
文件看起来：

```
<body>
  <%= render 'layouts/header' %>
  <div class="container">
    <%= render 'layouts/messages' %>
    <%= yield %>
    <%= render 'layouts/footer' %>
  </div>
</body>
```

Next we have to make the messages file.
下面制作信息文件。
Make a new file in **"app/views/layouts/_messages.html.erb"** and add:
新建一个文件`app/views/layouts/_messages.html.erb`，然后增加：

```erb
<% flash.each do |key, value| %>
  <div class="alert alert-<%= key %>">
    <a href="#" data-dismiss="alert" class="close">×</a>
      <ul>
        <li>
          <%= value %>
        </li>
      </ul>
  </div>
<% end %>
```

This will give us flash messages for the entire site.
这会为整个网站带来闪存信息。

<h3>Flash Messages For Devise Devise的闪存信息</h3>

For devise you need to override the way devise handles flash messages.
对于devise，需要改写devise处理闪存信息的方法。
Create a file called `devise_helper` in **"app/helpers/devise_helper.rb"**.
在创建一个app/helpers/devise_helper.rb文件。

Inside the file you have to create a method called devise_error_messages!, which is the name of the file that tells devise how to handle flash messages. 
在文件里面，必须创建一个`devise_error_messages!`的方法，该文件名告诉devise怎么处理闪存信息。

```ruby
module DeviseHelper
  def devise_error_messages!
    return '' if resource.errors.empty?

    messages = resource.errors.full_messages.map { |msg| content_tag(:li, msg) }.join
    html = <<-HTML
    <div class="alert alert-error alert-block"> <button type="button"
    class="close" data-dismiss="alert">x</button>
      #{messages}
    </div>
    HTML

    html.html_safe
  end
end
```

Next in your devise views you will have to define where you want the error messages to appear.
接下来在devise视图中，必须定义在哪里显示错误信息。
You will need to enter `<%= devise_error_messages! %>` within the devise pages.
需要在devise页面输入`<%= devise_error_messages! %>`。
An example is entering this within **"app/views/devise/registrations/.new.html.erb"** (The sign up page)
一个例子是在app/views/devise/registrations/.new.html.erb（注册页面）里面输入这个。

It should already be within the file, but you can move the code around to customize where it is shown.
它应该在文件里面了，但可以将代码移到定制它显示的地方。

<h3>CSS For Flash Messages 为闪存信息设置CSS</h3>

If you do not want to use the odd blue and yellow alerts that come default, I have set error and alert to have the same color and success and notice to have the same color.
如果不想使用默认里旧的蓝色和黄色警告，可以将错误和警告合并为一种颜色，成功和提示合并为同一种颜色。
I am using red for errors and alerts, and green for success and notice.
我则是将错误设置为红色，成功和提示设置为绿色。

Within my **"app/assets/stylesheets/custom.css.scss"** I have:
在app/assets/stylesheets/custom.css.scss：

```scss
/*flash*/
.alert-error {
    background-color: #f2dede;
    border-color: #eed3d7;
    color: #b94a48;
    text-align: left;
 }

.alert-alert {
    background-color: #f2dede;
    border-color: #eed3d7;
    color: #b94a48;
    text-align: left;
 }

.alert-success {
    background-color: #dff0d8;
    border-color: #d6e9c6;
    color: #468847;
    text-align: left;
 }

.alert-notice {
    background-color: #dff0d8;
    border-color: #d6e9c6;
    color: #468847;
    text-align: left;
 }
```
