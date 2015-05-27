As of version 1.7.0, CanCanCan now supports Strong Parameters and Rails 4 without controller workarounds.
到了1.7.0，CanCanCan已经支持强壮参数和Rails 4,无需控制器工作区了。
When using strong_parameters or Rails 4+, you have to sanitize inputs before saving the record, in actions such as `:create` and `:update`.
在Rails 4使用强壮参数时，需要在诸如创建和更新的动作保存记录之前消毒输入。
 
By default, CanCan will try to sanitize the input on `:create` and `:update` routes by seeing if your controller will respond to the following methods (in order):
默认情况下，通过检查控制器是否响应下面的方法（方法按顺序排列），CanCan将尝试消毒“创建”和“更新”路由的输入。

### By Action 通过动作

If you specify a `create_params` or `update_params` method, CanCan will run that method depending on the action you are performing.
如果指定了`create_params`或者`update_params`方法，CanCan将会根据运行的动作运行该方法。

```ruby
class ArticlesController < ApplicationController
  load_and_authorize_resource

  def create
    if @article.save
      # hurray
    else
      render :new
    end
  end

  def update
    if @article.update_attributes(update_params)
      # hurray
    else
      render :edit
    end
  end

  private

  def create_params
    params.require(:article).permit(:name, :email)
  end

  def update_params
    params.require(:article).permit(:name)
  end
end
```

### By Model Name 通过模型名称

If you follow the convention in rails for naming your param method after the applicable model's class `<model_name>_params` such as `article_params`, CanCanCan will automatically detect and run that params method.
如果依照Rails的惯例，在可用模型类`<model_name>_params`的后面来命名param方法，例如`article_params`，CanCanCan将自动侦测并运行该方法。

```ruby
class ArticlesController < ApplicationController
  load_and_authorize_resource

  def create
    if @article.save
      # hurray
    else
      render :new
    end
  end

  private

  def article_params
    params.require(:article).permit(:name)
  end
end
```

### By Static Method Name 通过静态方法名

CanCanCan also recognizes a static method name: `resource_params`, as a general param method name you can use to standardize on.
CanCanCan 也认可静态方法名`resource_params`，作为用于标准化的一般参数名称。

```ruby
class ArticlesController < ApplicationController
  load_and_authorize_resource

  def create
    if @article.save
      # hurray
    else
      render :new
    end
  end

  private

  def resource_params
    params.require(:article).permit(:name)
  end
end
```

### By Custom Method 通过自定义方法

Additionally, `load_and_authorize_resource` can now take a `param_method` option to specify a custom method in the controller to run to sanitize input.
另外，`load_and_authorize_resource`现在带有一个`param_method`可选来指定控制器的自定义方法，以便消毒输入。

```ruby
class ArticlesController < ApplicationController
  load_and_authorize_resource param_method: :my_sanitizer

  def create
    if @article.save
      # hurray
    else
      render :new
    end
  end

  private

  def my_sanitizer
    params.require(:article).permit(:name)
  end
end
```

### No Strong Parameters 没有强壮参数

No problem, if your controllers do not respond to any of the above methods, it will ignore and continue execution as normal.
没问题，如果控制器不响应上面的任何方法，它会忽略并像正常那样继续执行。

