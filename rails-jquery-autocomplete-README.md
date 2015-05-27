# vim: set filetype=markdown : 
## Status 状态

1. This is the officially maintained fork of [rails3-jquery-autocomplete](http://github.com/crowdint/rails3-jquery-autocomplete)
2.  All new features and support of Rails 4 and above will occur here going forward.
所有 Rails 4及以上版本的新特性和支持在这里继续保持。
 If you are still using Rails 3 please continue to use the previous fork.
如果仍然使用 Rails3,请继续使用之前的分支。
 No new features will be added there, but bug fixes & security patches will continue until Rails 3 is EOL.
新特性不会增加到那里，但臭虫修复和安全补丁会继续提供，直到Rails 3项目结束维护。


# rails-jquery-autocomplete
## 项目概述

[![Build Status](https://secure.travis-ci.org/bigtunacan/rails-jquery-autocomplete.png)](http://travis-ci.org/bigtunacan/rails-jquery-autocomplete) [![Gem Version](https://badge.fury.io/rb/rails-jquery-autocomplete.png)](http://badge.fury.io/rb/rails-jquery-autocomplete)

An easy way to use jQuery's autocomplete with Rails.

Supports both ActiveRecord, [mongoid](http://github.com/mongoid/mongoid), and [MongoMapper](https://github.com/jnunemaker/mongomapper).

Works with [Formtastic](http://github.com/justinfrench/formtastic)
and [SimpleForm](https://github.com/plataformatec/simple_form)

## ActiveRecord

You can find a [detailed example](http://github.com/crowdint/rails3-jquery-autocomplete-app)
on how to use this gem with ActiveRecord [here](http://github.com/crowdint/rails3-jquery-autocomplete-app).
在这里可以找到一个示例，展示如何使用这个gem。

## MongoID

You can find a [detailed example](http://github.com/crowdint/rails3-jquery-autocomplete-app/tree/mongoid)
on how to use this gem with MongoID [here](http://github.com/crowdint/rails3-jquery-autocomplete-app/tree/mongoid). (Same thing, different branch)

## Before you start  准备工作

Make sure your project is using jQuery-UI and the autocomplete widget before you continue.
确保项目使用jQuery-UI和autocomplete部件。

You can find more info about that here:
这里可以找到更多信息：

* http://jquery.com/
* http://jqueryui.com/demos/autocomplete/
* http://github.com/rails/jquery-ujs

I'd encourage you to understand how to use those 3 amazing tools before attempting to use this gem.
我鼓励你尝试使用这个软件包之前，先了解怎么使用这三个有用的工具。

<kinder:note> 一般rails项目已经有了`rails-jquery`，但还没有安装`jquery-ui`，因此需要安装：

  gem 'jquery-ui-rails'



## Installing  安装

Include the gem on your Gemfile
在Gemfile里面包含这个软件包：

    gem 'rails-jquery-autocomplete'

Install it
安装：

    bundle install

### Rails 4.x.x 生成

Run the generator
运行生成器安装：

    rails generate autocomplete:install

And include autocomplete-rails.js on your layouts
在布局中包含autocomplete-rails.js。

    javascript_include_tag "autocomplete-rails.js"

#### Upgrading from older versions 从老版本升级

If you are upgrading from a previous version, run the generator after installing to replace the javascript file.
如果正从之前的版本升级，在安装之后运行生成器来替换js文件：

    rails generate autocomplete:install

I'd recommend you do this every time you update to make sure you have the latest JS file.
推荐每次升级后都这么做。

#### Uncompressed Javascript file 未压缩js文件

If you want to make changes to the JS file, you can install the uncompressed version by running:
如果想更改js文件，可以运行这个来安装未压缩版本：

    rails generate autocomplete:uncompressed

### Rails 4 and higher 引用

Just add it to your app/assets/javascripts/application.js file
添加到app/assets/javascripts/application.js文件中：

    //= require jquery 
    //= require jquery_ujs
    //= require jquery-ui  # <kinder:note>这样引用开销太大，不如下一行：
    //= require jquery-ui/autocomplete  # <kinder:note> 小巧玲珑了许多。
    //= require autocomplete-rails

<kinder:note>当然，别忘了在css/application.css文件中引入
    */= require jquery-ui/autocomplete  # <kinder:note> 小巧玲珑了许多。


## Usage  用法

### Model Example  模型示例

Assuming you have a Brand model:
假设有一个Brand的模型

    class Brand < ActiveRecord::Base
    end

    create_table :brand do |t|
      t.column :name, :string
    end

### Controller  控制器

To set up the required action on your controller, all you have to do is call it with the class name and the method as in the following example:
要在控制器中设置所需的动作，只需要像下面的例子中用类名和方法来调用它即可：

    class ProductsController < Admin::BaseController
      autocomplete :brand, :name
    end

This will create an action `autocomplete_brand_name` on your controller, don't forget to add it on your routes file
这会在控制器中创建一个`autocomplete_brand_name`，不要忘了在路由文件中添加它：

    resources :products do
      get :autocomplete_brand_name, :on => :collection
    end

### Options  选项

#### :full => true  这是必须的

By default, the search starts from the beginning of the string you're searching for.
默认情况下，搜索开始于你要搜索的字符串的开端。
If you want to do a full search, set the _full_ parameter to true.
如果想完全搜索，需要将full参数设置为true。

    class ProductsController < Admin::BaseController
      autocomplete :brand, :name, :full => true
    end

The following terms would match the query 'un':
下面的条件都匹配查询‘un’：

* Luna
* Unacceptable
* Rerun

#### :full => false (default behavior)

Only the following terms mould match the query 'un':
只有下面这个条件会匹配查询‘un’：

* Unacceptable

#### :limit => 10 (default behavior)

By default your search result set is limited to the first 10 records. This can be overridden by specifying the limit option.
默认情况下，搜索结果集合仅限于开头的十个记录。
可以通过指定limit选项来改写。

#### :extra_data

By default, your search will only return the required columns from the database needed to populate your form, namely id and the column you are searching (name, in the above example).
默认情况下，搜索只会从数据库返回必要的栏目来生成表单，也就是id和搜索的栏目（在上面的例子中时name）。

Passing an array of attributes/column names to this option will fetch and return the specified data.
传递一个数组的属性/栏目名称到这个选项将能获取并返回指定的数据。

    class ProductsController < Admin::BaseController
      autocomplete :brand, :name, :extra_data => [:slogan]
    end

#### :display_value

If you want to display a different version of what you're looking for, you can use the :display_value option.
如果想要显示不同版本的搜索结果，可以设置这个选项。

This options receives a method name as the parameter, and that method will be called on the instance when displaying the results.
这个选项接受一个方法名作为参数，显示结果时该方法在实例中被调用。

    class Brand < ActiveRecord::Base
      def funky_method
        "#{self.name}.camelize"
      end
    end


    class ProductsController < Admin::BaseController
      autocomplete :brand, :name, :display_value => :funky_method
    end

In the example above, you will search by `name`, but the autocomplete list will display the result of `funky_method`
在上面的例子中，你会通过`name`来搜索，但自动完成列表将显示`funky_method`的结果。

This wouldn't really make much sense unless you use it with the "id_element" attribute.  (See below)
这不会真的起到作用，除非连用下面的`id_element`属性。

Only the object's id and the column you are searching on will be returned in JSON, so if your display_value method requires another parameter, make sure to fetch it with the :extra_data option
只有对象的id和搜索的栏目回在JSON中返回，所以如果显示值的方法要求另一个参数，要确保用`:extra_data`选项来取出。

#### :hstore

  Added option to support searching in hstore columns.
  添加选项到hstore栏里的支持搜索中。 

  Pass a hash with two keys: `:method` and `:key` with values: the hstore field name and the key of the hstore to search.
  传递一个带有两个key的散列：`:method`和`:key`，其值为hstore字段名称和要搜索的hstore的键。

  e.g `autocomplete :feature, :name, :hstore => {:method => 'name_translations', :key => 'en'}`

#### :scopes
  Added option to use scopes. Pass scopes in an array.
  添加选项来使用作用域。在数组中传给作用域。

  e.g `:scopes => [:scope1, :scope2]`

#### :column_name
   By default autocomplete uses method name as column name. 
   默认情况下，自动完成使用方法名作为栏名。
   Now it can be specified using column_name options
   可以使用`column_name`选项来制定了。

   `:column_name => 'name'`

#### json encoder
Autocomplete uses Yajl as JSON encoder/decoder, but you can specify your own
自动完成使用Yajl作为JSON编码解码器，但可以指定其他的：

    class ProductsController < Admin::BaseController
      autocomplete :brand, :name do |items|
         CustomJSON::Encoder.encode(items)
      end
    end


### View 视图

####  正常使用
On your view, all you have to do is include the attribute autocomplete on the text field using the url to the autocomplete action as the value.
在视图中，只需要在text字段中包含自动完成属性即可，可以使用一个到autocomplete的动作的地址作为值。

    form_for @product do |f|
      f.autocomplete_field :brand_name, autocomplete_brand_name_products_path
    end

This will generate an HTML tag that looks like:
这会产生一个HTML标签：

    <input type="text" data-autocomplete="products/autocomplete_brand_name">

If you are not using a FormBuilder (form_for) or you just want to include an autocomplete field without the form, you can use the *autocomplete_field_tag* helper.
如果没有使用FormBuilder（form_for），或者只想无需表单即可包含自动完成字段，可以使用`autocomplete_field_tag`辅助器：


    form_tag 'some/path'
      autocomplete_field_tag 'address', '', address_autocomplete_path, :size => 75
    end

#### Multiple Values Separated by Delimiter  用分隔符分开多个值

To generate an autocomplete input field that accepts multiple values separated by a given delimiter, add the `'data-delimiter'` and `:multiple` options:
要生成一个接受多个值（用给定分隔符独立）的自动完成输入字段，可以添加`data_delimiter`和`:multiple`选项：

    form_for @product do |f|
      f.autocomplete_field :brand_names, autocomplete_brand_name_products_path,
      'data-delimiter' => ',', :multiple => true
    end

NOTE: Setting the `:multiple` option to `true` will result in the chosen values being submitted as an array.
注意： 设置`:multiple`为`true`会导致所选的值被提交为一个数组。
Leaving this option off will result in the values being passed as a single string, with the values separated by your chosen delimiter.
关闭这个选项则会让值作为一个字符串传递，该值被所选的分隔符隔开。


#### Automatically focus on the first autocompleted item 自动聚焦第一个项目

To have the first item be automatically focused on when the autocomplete menu is shown, add the `'data-auto-focus'` option and set it to `true`.
自动完成菜单显示时，要让第一个项目自动获得聚焦，可以增加`data-auto-focus`选项并设置为`true`。

	form_for @product do |f|
		f.autocomplete_field :brand_names, autocomplete_brand_name_products_path,
		'data-auto-focus' => true
	end

Now your autocomplete code is unobtrusive, Rails style.
现在自动完成代码就变成Rails式的简洁了。


### Getting the object id  获得对象id

<kinder:note> 这个对象id不是数据源的对象的id，而是html元素的id。
If you need to use the id of the selected object, you can use the `id_element` attribute too:
如果需要使用所选对象的id，可以使用`id_element`属性。

    f.autocomplete_field :brand_name, autocomplete_brand_name_products_path, :id_element => '#some_element'

This will update the field with id `*#some_element` with the id of the selected object.
这回使用带有所选对象id的`*#some_element`的id更新字段。
The value for this option can be any jQuery selector.
这个选项的值可以是任何jQuery选择器。

### Changing destination element   更改目标元素

If you need to change destination element where the autocomplete box will be appended to, you can use the **:append_to** option which generates a **data-append-to** HTML attribute that is used in jQuery autocomplete as append_to attribute.
如果需要改变自动完成控件添加的目标元素，可以使用`append_to`选项，它会生成一个`data-append_to`HTML属性，作为append_to属性到jQuery自动完成中。

The :append_to option accepts a string containing jQuery selector for destination element:
`：append_to`选项接受一个字符串，为目标元素包含jQuery选择器。

    f.autocomplete_field :product_name, '/products/autocomplete_product_name', :append_to => "#product_modal"

The previous example would append the autocomplete box containing suggestions to element jQuery('#product_modal').
之前的示例会附加自动完成控件包含到元素jQuery('#product_modal')的暗示。
This is very useful on page where you use various z-indexes and you need to append the box to the topmost element, for example using modal window.
这在使用多个z-index的页面里非常有用，你需要附加控件到顶端的元素，比如使用模态窗口。
<kinder:note> 不懂。

### Sending extra search fields  发送额外的搜索字段

If you want to send extra fields from your form to the search action, you can use the **:fields** options which generates a **data-autocomplete-fields** HTML attribute.
如果想要从表单中发送额外的字段到搜索动作，可以使用`:fields`选项，可以产生一个`data-autocomplete-fields`html属性。

The :fields option accepts a hash where the keys represent the Ajax request parameter name and the values represent the jQuery selectors to retrieve the form elements to get the values:
`:field`选项接受一个散列，它的键表示Ajax请求参数的名称，值表示jQuery选择器来选取表单元素以获得值。

    f.autocomplete_field :product_name, '/products/autocomplete_product_name', :fields => {:brand_id => '#brand_element', :country => '#country_element'}

    class ProductsController < Admin::BaseController
      def autocomplete_product_name
        term = params[:term]
        brand_id = params[:brand_id]
        country = params[:country]
        products = Product.where('brand = ? AND country = ? AND name LIKE ?', brand_id, country, "%#{term}%").order(:name).all
        render :json => products.map { |product| {:id => product.id, :label => product.name, :value => product.name} }
      end
    end

### Getting extra object data  获得额外的对象数据

If you need to extra data about the selected object, you can use the `:update_elements` HTML attribute.
如果需要关于所选对象的额外数据，可以使用`:update_elements`HTML属性。

The :update_elements attribute accepts a hash where the keys represent the object attribute/column data to use to update and the values are jQuery selectors to retrieve the HTML element to update:
`update_elements`属性接收一个散列，键代表用于更新的对象属性/栏数据，值代表jQuery选择器接收HTML元素来更新。

    f.autocomplete_field :brand_name, autocomplete_brand_name_products_path, :update_elements => {:id => '#id_element', :slogan => '#some_other_element'}

    class ProductsController < Admin::BaseController
      autocomplete :brand, :name, :extra_data => [:slogan]
    end

The previous example would fetch the extra attribute slogan and update jQuery('#some_other_element') with the slogan value.
之前的示例会获取额外的属性slogna，并用slogan的值更新jQuery('#some_other_element')。

### Running custom code on selection  在选单中运行自定义代码

A javascript event named *railsAutocomplete.select* is fired on the input field when a value is selected from the autocomplete drop down.
当一个值从自动完成下拉菜单中被选择之后，输入框一个`railsAutocomplete.select`的javascript事件会被触发。
If you need to do something more complex than update fields with data, you can hook into this event, like so:
如果需要做些比更新输入框的数据更复杂的事情，可以钩住这个事件，类似于：

    $('#my_autocomplete_field').bind('railsAutocomplete.select', function(event, data){
      /* Do something here */
      alert(data.item.id);
    });


## Formtastic

If you are using [Formtastic](http://github.com/justinfrench/formtastic), you automatically get the *autocompleted_input* helper on *semantic_form_for*:

    semantic_form_for @product do |f|
      f.input :brand_name, :as => :autocomplete, :url => autocomplete_brand_name_products_path
    end

The only difference with the original helper is that you must specify the autocomplete url using the *:url* option.
和原有辅助器唯一不同的地方是，你需要使用`:url`选项指定自动完成地址。

## SimpleForm

If you want to use it with simple_form, all you have to do is use the
:as option on the input and set the autocomplete path with the :url
option.


    simple_form_for @product do |form|
      form.input :name
      form.input :brand_name, :url => autocomplete_brand_name_products_path, :as => :autocomplete

# Cucumber

I have created a step to test your autocomplete with Cucumber and Capybara, all you have to do is add the following lines to your *env.rb* file:

    require 'cucumber/autocomplete'

Then you'll have access to the following step:

    I choose "([^"]*)" in the autocomplete list

An example on how to use it:

    @javascript
    Scenario: Autocomplete
      Given the following brands exists:
        | name  |
        | Alpha |
        | Beta  |
        | Gamma |
      And I go to the home page
      And I fill in "Brand name" with "al"
      And I choose "Alpha" in the autocomplete list
      Then the "Brand name" field should contain "Alpha"

I have only tested this using Capybara, no idea if it works with something else, to see it in action, check the [example app](http://github.com/crowdint/rails3-jquery-autocomplete-app).

# Steak

I have created a helper to test your autocomplete with Steak and Capybara, all you have to do is add the following lines to your *acceptance_helper.rb* file:

    require 'steak/autocomplete'

Then you'll have access to the following helper:

    choose_autocomplete_result

An example on how to use it:

    scenario "Autocomplete" do
      lambda do
        Brand.create! [
          {:name => "Alpha"},
          {:name => "Beta"},
          {:name => "Gamma"}
        ]
      end.should change(Brand, :count).by(3)

      visit home_page
      fill_in "Brand name", :with => "al"
      choose_autocomplete_result "Alpha"
      find_field("Brand name").value.should include("Alpha")
    end

I have only tested this using Capybara, no idea if it works with something else.

# Development  开发

If you want to make changes to the gem, first install bundler 1.0.0:

    gem install bundler

And then, install all your dependencies:

    bundle install

## Running the test suite  运行测试套件

<strike>You need to have an instance of MongoDB running on your computer or all the mongo tests will fail miserably.</strike>

To run all the tests once, simply use

    rake test

while you're developing, it is recommended that you run

    bundle exec guard

to have the relevant test run every time you save a file.

## Integration tests   集成测试

If you make changes or add features to the jQuery part, please make sure
you write a cucumber test for it.

You can find an example Rails app on the *integration* folder.

You can run the integration tests with the cucumber command while on the
integration folder:

    cd integration
    rake db:migrate
    cucumber

## Where to test what

If you're making or tweaking a plugin (such as the formtastic plugin or simple\_form plugin), check out the simple\_form\_plugin\_test for an example of how to test it as part of the main `rake test` run.

Historically, plugins like these had been tested (shoddily) as part of the integration tests.
Feel free to remove them from the integration suite and move them into the main suite.
Your tests will run much faster, and there will be less likelihood of your feature breaking in the future.
Thanks!



# Thanks to  致谢

Everyone on [this list](https://github.com/crowdint/rails3-jquery-autocomplete/contributors)

# About the Author  作者

[Crowd Interactive](http://www.crowdint.com) is an American web design and development company that happens to work in Colima, Mexico.
We specialize in building and growing online retail stores. We don’t work with everyone – just companies we believe in. Call us today to see if there’s a fit.
Find more info [here](http://www.crowdint.com)!


