The `devise_error_messages!` method is overridden by adding a `devise_helper.rb` application helper.
`devise_error_messages!`方法可以通过添加一个`devise_helper.rb`程序辅助器来改写。

The standard implementation (below) can be cut and paste in then tweaked to your requirements.
下面的标准实现可被剪下粘帖，并根据需要进行调整。
It can also be helpful to define a convenience method to check for their presence in your views:
也用于定义一个方便的方法来检查视图中它们的存在

```ruby
module DeviseHelper
  def devise_error_messages!
    return "" if resource.errors.empty?

    messages = resource.errors.full_messages.map { |msg| content_tag(:li, msg) }.join
    sentence = I18n.t("errors.messages.not_saved",
                      :count => resource.errors.count,
                      :resource => resource.class.model_name.human.downcase)

    html = <<-HTML
    <div id="error_explanation">
      <h2>#{sentence}</h2>
      <ul>#{messages}</ul>
    </div>
    HTML

    html.html_safe
  end

  def devise_error_messages?
    resource.errors.empty? ? false : true
  end

end
```

More information at [this StackOverflow thread](http://stackoverflow.com/questions/4101641/rails-devise-handling-devise-error-messages).
更多信息在这里
