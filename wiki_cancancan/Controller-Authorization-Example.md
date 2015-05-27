CanCan provides a convenient `load_and_authorize_resource` method in the controller, but what exactly is this doing? 
CanCan 在控制器中提供了一个方便的`load_and_authorize_resource`方法，但这个方法实际上在干嘛？
It sets up a before filter for every action to handle the loading and authorization of the controller.
它为每个动作都设置了一个前置过滤器，来处理控制器的加载和授权。
Let's say we have a typical RESTful controller with that line at the top.
比如现在有一个表转风格的控制器，它的顶部就有这行代码：
 
```ruby
class ProjectsController < ApplicationController
  load_and_authorize_resource
  # ...
end
```

It will add a before filter that has this behavior for the actions if they exist.
它会为已有的控制器增加一个前置过滤器。
This means you do not need to put code below in your controller.
这意味着，无需要在控制器中放置以下代码：
 
```ruby
class ProjectsController < ApplicationController
  def index
    authorize! :index, Project
    @projects = Project.accessible_by(current_ability)
  end

  def show
    @project = Project.find(params[:id])
    authorize! :show, @project
  end

  def new
    @project = Project.new
    current_ability.attributes_for(:new, Project).each do |key, value|
      @project.send("#{key}=", value)
    end
    @project.attributes = params[:project]
    authorize! :new, @project
  end

  def create
    @project = Project.new
    current_ability.attributes_for(:create, Project).each do |key, value|
      @project.send("#{key}=", value)
    end
    @project.attributes = params[:project]
    authorize! :create, @project
  end

  def edit
    @project = Project.find(params[:id])
    authorize! :edit, @project
  end

  def update
    @project = Project.find(params[:id])
    authorize! :update, @project
  end

  def destroy
    @project = Project.find(params[:id])
    authorize! :destroy, @project
  end

  def some_other_action
    if params[:id]
      @project = Project.find(params[:id])
    else
      @projects = Project.accessible_by(current_ability)
    end
    authorize!(:some_other_action, @project || Project)
  end
end
```

The most complex behavior is inside the new and create actions.
最复杂的表现是新建和创建动作里面。
There it is setting some initial attribute values based on what the given user has permission to access.
这两个动作里，设置了一些初始化属性值，它们基于用户可访问的权限。
For example, if the user is only allowed to create projects where the "visible" attribute is true, then it would automatically set this upon building it.
比如，如果用户只允许创建“可见”属性为真的项目，那么，它就会自动设置该值，并构建起该项目。
 
See [[Authorizing Controller Actions]] for details on what options you can pass to the `load_and_authorize_resource`.
可传入`load_and_authorize_resource`里的选项详见《控制器动作的授权》。

# vim: set filetype=markdown : #
