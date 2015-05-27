Generally you only want to show new/edit/destroy links when the user has permission to perform that action. 
You can do so like this in the view.
当用户有权进行该操作时，一般只想显示“新增、编辑、删除”链接。

```rhtml
<% if can? :update, @project %>
  <%= link_to "Edit", edit_project_path(@project) %>
<% end %>
```

However if you find yourself repeating this pattern often you may want to add helper methods like this.
不过，如果经常重复这个模式，可能想像下面那样增加辅助器：

```ruby
# in ApplicationHelper
def show_link(object, content = "Show")
  link_to(content, object) if can?(:read, object)
end

def edit_link(object, content = "Edit")
  link_to(content, [:edit, object]) if can?(:update, object)
end

def destroy_link(object, content = "Destroy")
  link_to(content, object, :method => :delete, :confirm => "Are you sure?") if can?(:destroy, object)
end

def create_link(object, content = "New")
  if can?(:create, object)
    object_class = (object.kind_of?(Class) ? object : object.class)
    link_to(content, [:new, object_class.name.underscore.to_sym])
  end
end
```

Then a link is as simple as this.
然后连接就像下面这么简洁了：

```rhtml
<%= edit_link @project %>
```

I only recommend doing this if you see this pattern a lot in your application.
只有在程序中多次出现这个模式，才推荐这么做。
There are times when the view code is more complex where this doesn't fit well.
有时视图代码太复杂，就不适合这么做了。
