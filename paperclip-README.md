Paperclip
=========

[![Build Status](https://secure.travis-ci.org/thoughtbot/paperclip.png?branch=master)](http://travis-ci.org/thoughtbot/paperclip) [![Dependency Status](https://gemnasium.com/thoughtbot/paperclip.png?travis)](https://gemnasium.com/thoughtbot/paperclip) [![Code Climate](https://codeclimate.com/github/thoughtbot/paperclip.png)](https://codeclimate.com/github/thoughtbot/paperclip) [![Inline docs](http://inch-ci.org/github/thoughtbot/paperclip.png)](http://inch-ci.org/github/thoughtbot/paperclip) [![Security](https://hakiri.io/github/thoughtbot/paperclip/master.svg)](https://hakiri.io/github/thoughtbot/paperclip/master)

Paperclip is intended as an easy file attachment library for Active Record.
纸板是一个用于Active Record的易用的文件附件库。
The intent behind it was to keep setup as easy as possible and to treat files as much like other attributes as possible.
其背后的意图时是将设置变得尽可能的简单，尽量将文件作为其他属性看待。
This means they aren't saved to their final locations on disk, nor are they deleted if set to nil, until ActiveRecord::Base#save is called.
这意味着它们不会被保存到磁盘上最后的位置，也不会在被设置为nil时被删除（除非调用ActiveRecord::Base#save）。
It manages validations based on size and presence, if required.
如果需要，会基于文件大小和是否存在来管理验证。
It can transform its assigned image into thumbnails if needed, and the prerequisites are as simple as installing ImageMagick (which, for most modern Unix-based systems, is as easy as installing the right packages).
如果需要它可以将所指派的图片转换为缩略图，前提只是需要安装ImageMagick。
Attached files are saved to the filesystem and referenced in the browser by an easily understandable specification, which has sensible and useful defaults.
附件文件被保存到文件系统中，在浏览器里通过一个简易好懂的规范（有意义和有用的默认设置）引用。

See the documentation for `has_attached_file` in [`Paperclip::ClassMethods`](http://rubydoc.info/gems/paperclip/Paperclip/ClassMethods) for more detailed options.

The complete [RDoc](http://rdoc.info/gems/paperclip) is online.

---

Requirements  要求
------------

### Ruby and Rails

Paperclip now requires Ruby version **>= 1.9.2** and Rails version **>= 3.0** (Only if you're going to use Paperclip with Ruby on Rails.)
Paperclip 现在需要Ruby >=1.9.2以上， Rails版本3.0以上（如果你用的是RoR）

If you're still on Ruby 1.8.7 or Ruby on Rails 2.3.x, you can still use Paperclip 2.7.x with your project. 
Also, everything in this README might not apply to your version of Paperclip, and you should read [the README for version 2.7](http://rubydoc.info/gems/paperclip/2.7.0) instead.
如果还用Ruby 1.8.7或RoR2.3.x，可以使用paperclip 2.7.x版本。

### Image Processor   图片处理

[ImageMagick](http://www.imagemagick.org) must be installed and Paperclip must have access to it.
To ensure that it does, on your command line, run `which convert` (one of the ImageMagick utilities).
This will give you the path where that utility is installed.
For example, it might return `/usr/local/bin/convert`.

Then, in your environment config file, let Paperclip know to look there by adding that directory to its path.

In development mode, you might add this line to `config/environments/development.rb)`:

```ruby
Paperclip.options[:command_path] = "/usr/local/bin/"
```

If you're on Mac OS X, you'll want to run the following with Homebrew:

    brew install imagemagick

If you are dealing with pdf uploads or running the test suite, you'll also need GhostScript to be installed.
On Mac OS X, you can also install that using Homebrew:

    brew install gs

### `file`

The Unix [`file` command](http://en.wikipedia.org/wiki/File_(command)) is required for content type checking.
Unix`file`命令用来检查内容类型。

This utility isn't available in Windows, but comes bundled with Ruby [Devkit](https://github.com/oneclick/rubyinstaller/wiki/Development-Kit), so Windows users must make sure that the devkit is installed and added to system `PATH`.
这个工具在Windows系统没有提供，但它绑定在ruby Devkit中，所以Windows用户要确保安装devkit，并添加到系统的PATH中。

**Manual Installation**   手动安装

If you're using Windows 7+ as a development environment, you may need to install the `file.exe` application manually.
The `file spoofing` system in Paperclip 4+ relies on this; if you don't have it working, you'll receive `Validation failed:
Upload file has an extension that does not match its contents.` errors.


To manually install, you should perform the following:

> **Download & install `file` from [this URL](http://gnuwin32.sourceforge.net/packages/file.htm)**

To test, you can use the following:
![untitled](https://cloud.githubusercontent.com/assets/1104431/4524452/a1f8cce4-4d44-11e4-872e-17adb96f79c9.png)

Next, you need to integrate with your environment - preferrably through the `PATH` variable, or by changing your `config/environments/development.rb` file

**PATH**

    1. Click "Start"
    2. On "Computer", right-click and select "Properties"
    3. In properties, select "Advanced System Settings"
    4. Click the "Environment Variables" button
    5. Locate the "PATH" var - at the end, add the path to your newly installed `file.exe` (typically `C:\Program Files (x86)\GnuWin32\bin`)
    6. Restart any CMD shells you have open & see if it works

OR

**Environment**

    1. Open `config/environments/development.rb`
    2. Add the following line: `Paperclip.options[:command_path] = 'C:\Program Files (x86)\GnuWin32\bin'`
    3. Restart your Rails server

Either of these methods will give your Rails setup access to the `file.exe` functionality, this providing the ability to check the contents of a file (fixing the spoofing problem)

---

Installation 安装
------------

Paperclip is distributed as a gem, which is how it should be used in your app.

Include the gem in your Gemfile:
稳定版：

```ruby
gem "paperclip", "~> 4.2"
```

If you're still using Rails 2.3.x, you should do this instead:
Rails 2.3.x 系列：

```ruby
gem "paperclip", "~> 2.7"
```

Or, if you want to get the latest, you can get master from the main paperclip repository:
最新版本

```ruby
gem "paperclip", :git => "git://github.com/thoughtbot/paperclip.git"
```

If you're trying to use features that don't seem to be in the latest released gem, but are mentioned in this README, then you probably need to specify the master branch if you want to use them.
如果你尝试使用一些最新发行版软件包没有提供，但在这个README文件中提到的功能，你可能需要指定一个主分叉
This README is probably ahead of the latest released version, if you're reading it on GitHub.
如果你是在github里读到这个README，它的内容可能走在了最新发行版的前头。

For Non-Rails usage:
非Rails用法：

```ruby
class ModuleName < ActiveRecord::Base
  include Paperclip::Glue
  ...
end
```

---

Quick Start  快速开始
-----------

### Models  在模型中

**Rails 3**

```ruby
class User < ActiveRecord::Base
  attr_accessible :avatar
  has_attached_file :avatar, :styles => { :medium => "300x300>", :thumb => "100x100>" }, :default_url => "/images/:style/missing.png"
  validates_attachment_content_type :avatar, :content_type => /\Aimage\/.*\Z/
end
```

**Rails 4**

```ruby
class User < ActiveRecord::Base
  has_attached_file :avatar, :styles => { :medium => "300x300>", :thumb => "100x100>" }, :default_url => "/images/:style/missing.png"
  validates_attachment_file_name :avatar, :matches => [/png\Z/, /jpe?g\Z/]
  validates_attachment_content_type :avatar, :content_type => /\Aimage\/.*\Z/
end
```

### Migrations  在迁移中

```ruby
class AddAvatarColumnsToUsers < ActiveRecord::Migration
  def self.up
    add_attachment :users, :avatar
  end

  def self.down
    remove_attachment :users, :avatar
  end
end
```

(Or you can use migration generator: `rails generate paperclip user avatar`)
或者可以使用迁移生成器

### Edit and New Views  编辑和新建视图（表单）

```erb
<%= form_for @user, :url => users_path, :html => { :multipart => true } do |form| %>
  <%= form.file_field :avatar %>
<% end %>
```

### Edit and New Views with Simple Form   使用Simple Form编辑视图
```erb
<%= simple_form_for @user, url: users_path do |form| %>
  <%= form.input :avatar, as: :file %>
<% end %>
```

### Controller  控制器

**Rails 3**

```ruby
def create
  @user = User.create( params[:user] )
end
```

**Rails 4**

```ruby
def create
  @user = User.create( user_params )
end

private

# Use strong_parameters for attribute whitelisting
# Be sure to update your create() and update() controller methods.

def user_params
  params.require(:user).permit(:avatar)   # <kinder:note> 健壮参数
end
```

### Show View    show视图

```erb
<%= image_tag @user.avatar.url %>
<%= image_tag @user.avatar.url(:medium) %>
<%= image_tag @user.avatar.url(:thumb) %>
```

### Deleting an Attachment   删除附件

Set the attribute to `nil` and save.

```ruby
@user.avatar = nil
@user.save
```
---

Usage  用法
-----

The basics of paperclip are quite simple:
基本用法十分简单：
Declare that your model has an attachment with the `has_attached_file` method, and give it a name.
在模型中声明一个附件方法`has_attached_file`，并给定一个名称。

Paperclip will wrap up to four attributes (all prefixed with that attachment's name, so you can have multiple attachments per model if you wish) and give them a friendly front end.
Paperclip 将会包装四个属性（全部以附件的名称为前缀，如果喜欢可以给每个模型安排多个附件），并给出一个友好的前端。

These attributes are:
属性有：

* `<attachment>_file_name`
* `<attachment>_file_size`
* `<attachment>_content_type`
* `<attachment>_updated_at`

By default, only `<attachment>_file_name` is required for paperclip to operate.
默认情况下，只有`<attachment>_file_name`是必须提供来操作的。
You'll need to add `<attachment>_content_type` in case you want to use content type validation.
如果需要验证内容类型，可以添加`<attachment>_content_type`。

More information about the options to `has_attached_file` is available in the
documentation of [`Paperclip::ClassMethods`](http://rubydoc.info/gems/paperclip/Paperclip/ClassMethods).
关于`has_attached_file`选项的更多信息在文件中。

Validations 验证
-----------

For validations, Paperclip introduces several validators to validate your attachment:
对于验证，Paperclip 引入了几种验证器来验证附件

* `AttachmentContentTypeValidator`  文件类型
* `AttachmentPresenceValidator`     文件存在
* `AttachmentSizeValidator`         文件大小

Example Usage:
用法示例：

```ruby
validates :avatar, :attachment_presence => true
validates_with AttachmentPresenceValidator, :attributes => :avatar
validates_with AttachmentSizeValidator, :attributes => :avatar, :less_than => 1.megabytes

```

Validators can also be defined using the old helper style:
验证器可以用辅助器的方式来定义：

* `validates_attachment_presence`
* `validates_attachment_content_type`
* `validates_attachment_size`

Example Usage:
用法示例：

```ruby
validates_attachment_presence :avatar
```

Lastly, you can also define multiple validations on a single attachment using `validates_attachment`:
最后，你也可以使用`validates_attachment`来为单独一个附件定义多个验证器。

```ruby
validates_attachment :avatar, 
   :presence => true,
  :content_type => { :content_type => "image/jpeg" },
  :size => { :in => 0..10.kilobytes }
```

_NOTE: Post processing will not even *start* if the attachment is not valid
according to the validations. Your callbacks and processors will *only* be
called with valid attachments._
注意，如果附件没有通过验证，则提交过程(`post_process`)不会开始。回调和处理器将只能和有效附件一起调用。

```ruby
class Message < ActiveRecord::Base
  has_attached_file :asset, styles: {thumb: "100x100#"}

  before_post_process :skip_for_audio

  def skip_for_audio
    ! %w(audio/ogg application/ogg).include?(asset_content_type)
  end
end
```

If you have other validations that depend on assignment order, the recommended course of action is to prevent the assignment of the attachment until afterwards, then assign manually:
如果存在其他的依赖任务顺序的验证，推荐的动作方针时，阻止附件的分配，直到手工分配。

```ruby
class Book < ActiveRecord::Base
  has_attached_file :document, styles: {thumbnail: "60x60#"}
  validates_attachment :document, content_type: { content_type: "application/pdf" }
  validates_something_else # Other validations that conflict with Paperclip's
                           # 其他和Paperclip的验证相冲突的验证
end

class BooksController < ApplicationController
  def create
    @book = Book.new(book_params)
    @book.document = params[:book][:document]
    @book.save
    respond_with @book
  end

  private

  def book_params
    params.require(:book).permit(:title, :author)
  end
end
```

**A note on `content_type` validations and security**
`content_type`验证和安全的提示：

You should ensure that you validate files to be only those MIME types you explicitly want to support.
你应该确保验证文件是属于那些你明确想要支持的MIME类型。
 If you don't, you could be open to <a href="https://www.owasp.org/index.php/Testing_for_Stored_Cross_site_scripting_(OWASP-DV-002)">XSS attacks</a> if a user uploads a file with a malicious HTML payload.
否则，会开放了XSS攻击，如果用户上传了具有恶意HTML负载的文件。

If you're only interested in images, restrict your allowed content_types to image-y ones:
如果只限制于图片，可以限制`content_types`为其中一个图片格式。

```ruby
validates_attachment :avatar,
  :content_type => { :content_type => ["image/jpeg", "image/gif", "image/png"] }
```

`Paperclip::ContentTypeDetector` will attempt to match a file's extension to an inferred content_type, regardless of the actual contents of the file.
`Paperclip::ContentTypeDetector`会尝试根据文件的实际内容，去匹配文件的扩建名和相关的内容类型。

---

Security Validations  安全验证
====================

Thanks to a report from [Egor Homakov](http://homakov.blogspot.com/) we have taken steps to prevent people from spoofing Content-Types and getting data you weren't expecting onto your server.
十分感谢Egor homakov的报告，我们已经采取步骤预防人们欺骗内容类型（Content-Types）并取得服务器上的未经许可的数据。

NOTE: Starting at version 4.0.0, all attachments are *required* to include a content_type validation, a file_name validation, or to explicitly state that they're not going to have either.
注意，从4.0.0版本开始，所有的附件都必须包含一个内容类型验证、文件名验证，否则需要明确表明它们两者都不需要。

*Paperclip will raise an error* if you do not do this.
*如果没有做到这些，Paperclip会抛出一个错误*

```ruby
class ActiveRecord::Base
  has_attached_file :avatar
  # Validate content type  验证内容类型
  validates_attachment_content_type :avatar, :content_type => /\Aimage/
  # Validate filename   验证文件名
  validates_attachment_file_name :avatar, :matches => [/png\Z/, /jpe?g\Z/]
  # Explicitly do not validate  明确表明不验证
  do_not_validate_attachment_file_type :avatar
end
```

This keeps Paperclip secure-by-default, and will prevent people trying to mess with your filesystem.
这让Paperclip保持安全，并阻止人们尝试弄乱你的文件系统。

NOTE: Also starting at version 4.0.0, Paperclip has another validation that cannot be turned off.
注意，也是开始于4.0.0版本，Paperclip关闭了另一个验证。
This validation will prevent content type spoofing.
这个验证会阻止内容类型欺骗。
That is, uploading a PHP document (for example) as part of the EXIF tags of a well-formed JPEG.
具体来说，上传一个php文档作为JPEG图片的EXIF标签的一部分。
This check is limited to the media type (the first part of the MIME type, so, 'text' in 'text/plain').
这个检查仅限于媒体类型（MIME类型的第一部分，因此，是'text/plain'的'TEXT'）
This will prevent HTML documents from being uploaded as JPEGs, but will not prevent GIFs from being uploaded with a .jpg extension.
这会阻止HTML文档作为JPEG来上传，但不能阻止GIF文件附加`.jpg`扩展来上传。
This validation will only add validation errors to the form.
这个验证将只添加一个验证错误到表单上。
It will not cause Errors to be raised.
不会引发并抛出错误。

This can sometimes cause false validation errors in applications that use custom file extensions.
这有时在程序中使用自定义文件扩展时会导致验证失败的错误。
In these cases you may wish to add your custom extension to the list of file extensions allowed for your mime type configured by the mime-types gem:
这种例子里，你可能想添加自定义的扩展到文件扩展列表中，允许通过mime-types软件包让mime类型进行配置。

```ruby
# Allow ".foo" as an extension for files with the mime type "text/plain".
# 允许 ".foo" 作为"text/plain"的mime类型的扩展。
text_plain = MIME::Types["text/plain"].first
text_plain.extensions << "foo"
MIME::Types.index_extensions text_plain
```

---

Defaults  默认
--------
Global defaults for all your paperclip attachments can be defined by changing the `Paperclip::Attachment.default_options` Hash, this can be useful for setting your default storage settings per example so you won't have to define them in every has_attached_file definition.
所有Paperclip附件的全局设置可通过改变`Paperclip::Attachment.default_options`散列来更改。这在设置每个例子的默认存储设置中十分有用，因此不需要在每个`has_attached_file`定义中进行定义。

If you're using Rails you can define a Hash with default options in config/application.rb or in any of the config/environments/*.rb files on config.paperclip_defaults, these will get merged into `Paperclip::Attachment.default_options` as your Rails app boots.
如果你正在使用Rails，可以在config/application.rb或者任何config/environments/*.rb文件里，使用config.paperclip_defaults默认选项定义散列，这些配置会被合并到`Paperclip::Attachment.default_options`中，作为Rils程序的启动选项。
An example:
例如：

```ruby
module YourApp
  class Application < Rails::Application
    # Other code...

    config.paperclip_defaults = {:storage => :fog, :fog_credentials => {:provider => "Local", :local_root => "#{Rails.root}/public"}, :fog_directory => "", :fog_host => "localhost"}
  end
end
```

Another option is to directly modify the `Paperclip::Attachment.default_options` Hash, this method works for non-Rails applications or is an option if you prefer to place the Paperclip default settings in an initializer.
另一个方案是直接修改`Paperclip::Attachment.default_options`散列，这个方法适用于非Rails程序，或者你喜欢在启动器中替换默认设置。

An example Rails initializer would look something like this:
Rails 启动器的例子：

```ruby
Paperclip::Attachment.default_options[:storage] = :fog
Paperclip::Attachment.default_options[:fog_credentials] = {:provider => "Local", :local_root => "#{Rails.root}/public"}
Paperclip::Attachment.default_options[:fog_directory] = ""
Paperclip::Attachment.default_options[:fog_host] = "http://localhost:3000"
```
---

Migrations  迁移
----------

Paperclip defines several migration methods which can be used to create necessary columns in your model. 
There are two types of method:
Paperclip 定义了几个迁移方法，可用于创建模型必要的数据栏。
有两种类型：

### Table Definition  表定义

```ruby
class AddAttachmentToUsers < ActiveRecord::Migration
  def self.up
    create_table :users do |t|
      t.attachment :avatar
    end
  end
end
```

If you're using Rails 3.2 or newer, this method works in `change` method as well:
如果你使用了 Rails 3.2 或更新的版本，可以用'change'方法：

```ruby
class AddAttachmentToUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.attachment :avatar
    end
  end
end
```

### Schema Definition  纲要定义

```ruby
class AddAttachmentToUsers < ActiveRecord::Migration
  def self.up
    add_attachment :users, :avatar
  end

  def self.down
    remove_attachment :users, :avatar
  end
end
```

If you're using Rails 3.2 or newer, you only need `add_attachment` in your `change` method:
如果你使用了 Rails 3.2 或更新的版本，可以用'add_attachment'方法：

```ruby
class AddAttachmentToUsers < ActiveRecord::Migration
  def change
    add_attachment :users, :avatar
  end
end
```

### Vintage syntax  Vintage语法

Vintage syntax (such as `t.has_attached_file` and `drop_attached_file`) are still supported in Paperclip 3.x, but you're advised to update those migration files to use this new syntax.
Vintage语法（例如`t.has_attached_file`和`drop_attached_file`）在 Rails 3 中仍被支持，但建议使用新语法来更新那些迁移。

---

Storage  存储
-------

Paperclip ships with 3 storage adapters:
Paperclip使用3种存储适配器来装载：

* File Storage   文件存储
* S3 Storage (via `aws-sdk`)   亚马逊存储
* Fog Storage   Fog存储

If you would like to use Paperclip with another storage, you can install these gems along side with Paperclip:
如果你想让Paperclip使用其他的存储方式，可以加装这些软件包：

* [paperclip-azure-storage](https://github.com/gmontard/paperclip-azure-storage)
* [paperclip-dropbox](https://github.com/janko-m/paperclip-dropbox)

### Understanding Storage  理解存储

The files that are assigned as attachments are, by default, placed in the directory specified by the `:path` option to `has_attached_file`.
那些作为附件的文件，默认情况下会放在由`has_attached_file`的':path'选项指定的目录中。
By default, this location is `:rails_root/public/system/:class/:attachment/:id_partition/:style/:filename`.
默认位置是`:rails_root/public/system/:class/:attachment/:id_partition/:style/:filename`。

This location was chosen because on standard Capistrano deployments, the `public/system` directory is symlinked to the app's shared directory, meaning it will survive between deployments.
选择这个位置是为了标准的Capistrano部署，`public/system`目录会链接到程序的共享目录，这就意味着它会在部署之间一直存在。
For example, using that `:path`, you may have a file at `/data/myapp/releases/20081229172410/public/system/users/avatar/000/000/013/small/my_pic.png` 
比如，使用那个`:path`，你可能有个文件在`/data/myapp/releases/20081229172410/public/system/users/avatar/000/000/013/small/my_pic.png`。

_**NOTE**: This is a change from previous versions of Paperclip, but is overall a safer choice for the default file store._ 
注意：这和Paperclip之前的版本并不一样，但对于默认存储来说，这是一个更安全的选择。
You may also choose to store your files using Amazon's S3 service.
你可能也会选择亚马逊的S3服务来存储文件。
To do so, include the `aws-sdk` gem in your Gemfile:
要这么做，可在Gimfile里包含`aws-sdk`软件包：

```ruby
gem 'aws-sdk', '~> 1.5.7'
```

And then you can specify using S3 from `has_attached_file`.
然后可以在`has_attached_file`指定使用S3。
You can find more information about configuring and using S3 storage in
使用S3存储的更多配置信息参见：
[the `Paperclip::Storage::S3` documentation](http://rubydoc.info/gems/paperclip/Paperclip/Storage/S3).

Files on the local filesystem (and in the Rails app's public directory) will be available to the internet at large.
本地文件系统的文件将可被互联网所共享。
If you require access control, it's possible to place your files in a different location.
如果你需要访问控制，就需要在不同的位置存放这些文件。
You will need to change both the `:path` and `:url` options in order to make sure the files are unavailable to the public.
需要更改`:path`和`:url`选项，以便确保文件不在public中。
Both `:path` and `:url` allow the same set of interpolated variables.
`:path`和`:url`都允许同样的插入变量集合。

---

Post Processing 提交处理
---------------

Paperclip supports an extensible selection of post-processors.
Paperclip 支持可扩展的提交处理器的选择。
When you define a set of styles for an attachment, by default it is expected that those "styles" are actually "thumbnails".
当你为一个附件定了一组风格，默认下它期望那些风格为“thmbnails”。
However, you can do much more than just thumbnail images.
不过，你可以比缩略图做得更多。
By defining a subclass of Paperclip::Processor, you can perform any processing you want on the files that are attached.
通过定义Paperclip::Processor的子类，可以对附件操作任何你愿意的处理。
Any file in your Rails app's `lib/paperclip` and `lib/paperclip_processors` directories is automatically loaded by paperclip, allowing you to easily define custom processors.
在Rails程序的`lib/paperclip`和`lib/paperclip_processors`目录里的任何文件，会自动被paperclip加载，允许你方便地定义自定义处理器。
You can specify a processor with the :processors option to `has_attached_file`:
可以使用`has_attached_file`的:processors选项指定一个处理器。

```ruby
has_attached_file :scan, :styles => { :text => { :quality => :better } },
                         :processors => [:ocr]
```

This would load the hypothetical class Paperclip::Ocr, which would have the hash "{ :quality => :better }" passed to it along with the uploaded file.
这会加载虚拟类Paperclip::Ocr，它会将散列"{ :quality => :better }"和上传的文件一起传递。
For more information about defining processors, see Paperclip::Processor.
定义处理器的更多信息参见Paperclip::Process。

The default processor is Paperclip::Thumbnail.
默认处理器是Paperclip::Thumbnail。
For backwards compatibility reasons, you can pass a single geometry string or an array containing a geometry and a format, which the file will be converted to, like so:
为了向后兼容，可以传递一个长宽字符串或一个包含长宽和格式的数组，这样文件将被转化。类似于：

```ruby
has_attached_file :avatar, :styles => { :thumb => ["32x32#", :png] }
```

This will convert the "thumb" style to a 32x32 square in png format, regardless of what was uploaded.
这会将缩略图风格转为32×32正方形的png格式，不管上传的文件是什么。
If the format is not specified, it is kept the same (i.e.  jpgs will remain jpgs). 
如果没有指定格式，则保持原有的格式名称（比如jpg会保持为jpg）。
For more information on the accepted style formats, see
[here](http://www.imagemagick.org/script/command-line-processing.php#geometry).
更多可接受的风格格式的信息参见这个链接。

Multiple processors can be specified, and they will be invoked in the order they are defined in the :processors array.
多处理器可被指定，它们会按在:processors数组里定义的顺序被调用。
Each successive processor will be given the result of the previous processor's execution.
每个连续的处理器将被输入前面处理器的执行结果。
All processors will receive the same parameters, which are what you define in the :styles hash.
所有的处理器将接受同样的参数，这些参数是你在:styles散列里定义的。

For example, assuming we had this definition:
比如，假设我们有这些定义：

```ruby
has_attached_file :scan, :styles => { :text => { :quality => :better } },
                         :processors => [:rotator, :ocr]
```

then both the :rotator processor and the :ocr processor would receive the options "{ :quality => :better }".
rotator处理器和ocr处理器都将接受选项"{ :quality => :better }"。
This parameter may not mean anything to one or more or the processors, and they are expected to ignore it.
这个参数可能对一个或多个处理器没有意义，它们会被忽略。


_NOTE: Because processors operate by turning the original attachment into the styles, no processors will be run if there are no styles defined._
注意：因为处理器通过将原始附加转换为特定风格进行操作；所以如果没有定义好风格则没有处理器会运行。

If you're interested in caching your thumbnail's width, height and size in the database, take a look at the [paperclip-meta](https://github.com/teeparham/paperclip-meta) gem.
如果你对缓存缩略图的宽度、长度和大小感兴趣，可以看看paperclip-meta软件包。

Also, if you're interested in generating the thumbnail on-the-fly, you might want to look into the [attachment_on_the_fly](https://github.com/drpentode/Attachment-on-the-Fly) gem.
还有，如果你都快速生成缩略图感兴趣，可以看看快速附件软件包。

---

Events  事件和回调
------

Before and after the Post Processing step, Paperclip calls back to the model with a few callbacks, allowing the model to change or cancel the processing step.
在提交处理这一步前后，Paperclip会回调一些模型回调，允许模型改变或取消处理步骤。
The callbacks are `before_post_process` and `after_post_process` (which are called before and after the processing of each attachment), and the attachment-specific `before_<attachment>_post_process` and `after_<attachment>_post_process`.
回调可以在处理之前和处理之后（处理每个附件的之前和之后），以及在特定附件的处理之前`before_<attachment>_post_process`和处理之后`after_<attachment>_post_process`。
The callbacks are intended to be as close to normal ActiveRecord callbacks as possible, so if you return false (specifically \- returning nil is not the same) in a `before_filter`, the post processing step will halt.
回调会尽量接近普通的ActiveRecord回调，因此如果在前置过滤器返回false（特别是，返回nil是不同的），提交过程这一步将被中断。
Returning false in an `after_filter` will not halt anything, but you can access the model and the attachment if necessary.
在后置过滤器返回false不会中断提交，但如果必要可以访问模型和附件。

_NOTE: Post processing will not even *start* if the attachment is not valid according to the validations. 
Your callbacks and processors will *only* be called with valid attachments._
注意，如果附件没有通过验证，则提交过程(`post_process`)不会开始。回调和处理器将只能和有效附件一起调用。

```ruby
class Message < ActiveRecord::Base
  has_attached_file :asset, styles: {thumb: "100x100#"}

  before_post_process :skip_for_audio

  def skip_for_audio
    ! %w(audio/ogg application/ogg).include?(asset_content_type)
  end
end
```

---

URI Obfuscation  地址迷惑
---------------

Paperclip has an interpolation called `:hash` for obfuscating filenames of publicly-available files.
Paperclip有一个叫做`:hash`插补，用于迷惑公共文件夹里文件的文件名。

Example Usage:
用法举例：

```ruby
has_attached_file :avatar, {
    :url => "/system/:hash.:extension",
    :hash_secret => "longSecretString"
}
```


The `:hash` interpolation will be replaced with a unique hash made up of whatever is specified in `:hash_data`.
`:hash`插补将会被一个唯一的散列所替换，无论在`:hash_data`中指定了什么，都会被补足。
The default value for `:hash_data` is `":class/:attachment/:id/:style/:updated_at"`.
默认的`:hash_data`值是`":class/:attachment/:id/:style/:updated_at"`


`:hash_secret` is required, an exception will be raised if `:hash` is used without `:hash_secret` present.
`:hash_secret`是必要参数，如果用了`:hash`却没有`:hash_secret`则会抛出异常。
For more on this feature read the author's own explanation. [https://github.com/thoughtbot/paperclip/pull/416](https://github.com/thoughtbot/paperclip/pull/416)
这个特性的更多信息请阅读作者自己的解释。

MD5 Checksum / Fingerprint  文件指纹
-------

A MD5 checksum of the original file assigned will be placed in the model if it has an attribute named fingerprint.
如果有“指纹”属性，原始文件赋值的MD5校验和将被存放在模型中。
 Following the user model migration example above, the migration would look like the following.
沿用上面的user模型迁移的例子，迁移应该像：

```ruby
class AddAvatarFingerprintColumnToUser < ActiveRecord::Migration
  def self.up
    add_column :users, :avatar_fingerprint, :string
  end

  def self.down
    remove_column :users, :avatar_fingerprint
  end
end
```

File Preservation for Soft-Delete  文件保存以便软删除
-------

An option is available to preserve attachments in order to play nicely with soft-deleted models. (acts_as_paranoid, paranoia, etc.)
一个选项可用来保存附件，以便使用软删除模型（就像`acts_as_paranoid`, `paranoia`那样）

```ruby
has_attached_file :some_attachment, {
    :preserve_files => "true",
}
```

This will prevent ```some_attachment``` from being wiped out when the model gets destroyed, so it will still exist when the object is restored later.
这会阻止附件在模型删除时被清除，因此它会在对象恢复时依旧存在。

---

Custom Attachment Processors  自定义附件处理器
-------

Custom attachment processors can be implemented and their only requirement is to inherit from `Paperclip::Processor` (see `lib/paperclip/processor.rb`).
自定义附件处理器可以实现，唯一的要求是继承`Paperclip::Processor`（参见`lib/paperclip/processor.rb`）。

For example, when `:styles` are specified for an image attachment, the thumbnail processor (see `lib/paperclip/thumbnail.rb`) is loaded without having to specify it as a `:processor` parameter to `has_attached_file`.
比如，当`:styles`被一个图片附件所指定，缩略图处理器会被加载，无需指定一个`:processor`参数到`has_attached_file`中。
 When any other processor is defined it must be called out in the `:processors` parameter if it is to be applied to the attachment.
定义了其他处理器的时候，如果要应用到附件中，则还需要在`:processors`参数中调用。
 The thumbnail processor uses the imagemagick `convert` command to do the work of resizing image thumbnails.
缩略图处理器使用imagemagick的`convert`命令来生成图片缩略图。
 It would be easy to create a custom processor that watermarks an image using imagemagick's `composite` command.
很容易使用imagemagick的`composite`命令创建一个盖水印的自定义处理器。
 Following the implementation pattern of the thumbnail processor would be a way to implement a watermark processor.
跟着缩略图处理的实现模型，就可以实现水印处理器了。
 All kinds of attachment processors can be created; a few utility examples would be compression and encryption processors.
可以创建所有类型的附件处理器，一些有用的例子是压缩和加密处理器。

---

Dynamic Configuration  动态配置
---------------------

Callable objects (lambdas, Procs) can be used in a number of places for dynamic configuration throughout Paperclip.
可调用对象（lambdas，Procs）可用于一些地方来动态配置Paperclip。
 This strategy exists in a number of components of the library but is most significant in the possibilities for allowing custom styles and processors to be applied for specific model instances, rather than applying defined styles and processors across all instances.
这个策略存在于一些库的组建，可能最有意义还是允许自定义风格和处理器，然后应用于特定的模型示例，而不是对所有实例应用定义好的风格和处理器。


### Dynamic Styles:  动态风格

Imagine a user model that had different styles based on the role of the user.
想想一个用户模型，用户们因为角色不同拥有不同的风格。
Perhaps some users are bosses (e.g. a User model instance responds to #boss?) and merit a bigger avatar thumbnail than regular users.
可能一些用户是老板，因此头像会比普通用户更大些。
The configuration to determine what style parameters are to be used based on the user role might look as follows where a boss will receive a `300x300` thumbnail otherwise a `100x100` thumbnail will be created.
基于用户的角色决定使用什么风格参数的配置，看起来类似于下面，老板将接受一个`300×300`的缩略图，否则创建一个`100×100`的缩略图。

```ruby
class User < ActiveRecord::Base
  has_attached_file :avatar, :styles => lambda { |attachment| { :thumb => (attachment.instance.boss? ? "300x300>" : "100x100>") } }
end
```

### Dynamic Processors: 动态处理器

Another contrived example is a user model that is aware of which file processors should be applied to it (beyond the implied `thumbnail` processor invoked when `:styles` are defined).
另一个可用的例子是用户模型，可以知道应用哪个文件处理器（这就超过了`:style`定义之后调用缩略图处理器了）。
Perhaps we have a watermark processor available and it is only used on the avatars of certain models.
可能有了水印处理器，只用于特定模型的头像。

 The configuration for this might be where the instance is queried for which processors should be applied to it.
这个配置让处理器可以应用到所查询的实例

Presumably some users might return `[:thumbnail, :watermark]` for its processors, where a defined `watermark` processor is invoked after the `thumbnail` processor already defined by Paperclip.
可能会为有些用户返回处理器`[:thumbnail, :watermark]`，在缩略图处理器之后调用一个定义好的水印处理器，或者Paperclip已经定义好了。


```ruby
class User < ActiveRecord::Base
  has_attached_file :avatar, :processors => lambda { |instance| instance.processors }
  attr_accessor :processors
end
```

---

Logging 日志
----------

By default Paperclip outputs logging according to your logger level.
默认下Paperclip根据日志器的级别输出日志。
If you want to disable logging (e.g. during testing) add this in to your environment's configuration:
如果需要关闭日志登记（比如在测试期间），可将这一行添加到环境配置中：

```ruby
Your::Application.configure do
...
  Paperclip.options[:log] = false
...
end
```

More information in the [rdocs](http://rdoc.info/github/thoughtbot/paperclip/Paperclip.options)
更多信息参见rdocs。

---

Deployment  部署
----------

Paperclip is aware of new attachment styles you have added in previous deploys.
Paperclip 知道在前面的部署中你添加的新的附件风格。
The only thing you should do after each deployment is to call `rake paperclip:refresh:missing_styles`.
部署之后要做的是调用`rake paperclip:refresh:missing_styles`。

 It will store current attachment styles in `RAILS_ROOT/public/system/paperclip_attachments.yml` by default.
默认情况下它会将当前附件风格存储在`RAILS_ROOT/public/system/paperclip_attachments.yml`中。
You can change it by:
可以通过这一行来改变：

```ruby
Paperclip.registered_attachments_styles_path = '/tmp/config/paperclip_attachments.yml'
```

Here is an example for Capistrano:
下面是一个示例

```ruby
namespace :deploy do
  desc "build missing paperclip styles"
  task :build_missing_paperclip_styles do
    on roles(:app) do
      execute "cd #{current_path}; RAILS_ENV=production bundle exec rake paperclip:refresh:missing_styles"
    end
  end
end

after("deploy:compile_assets", "deploy:build_missing_paperclip_styles")
```

Now you don't have to remember to refresh thumbnails in production every time you add a new style.
这样你就不会每次在生成环境中添加新风格之后忘记刷新缩略图了。

Unfortunately it does not work with dynamic styles - it just ignores them.
不幸的是，动态风格不能运行——它会忽略它们。

If you already have a working app and don't want `rake paperclip:refresh:missing_styles` to refresh old pictures, you need to tell Paperclip about existing styles.
如果你的程序正在运行，不想``来刷新旧的图片，那就需要告诉Paperclip已有的风格。
Simply create a `paperclip_attachments.yml` file by hand.
只需要手工创建一个`paperclip_attachments.yml`文件。
For example:
比如：

```ruby
class User < ActiveRecord::Base
  has_attached_file :avatar, :styles => {:thumb => 'x100', :croppable => '600x600>', :big => '1000x1000>'}
end

class Book < ActiveRecord::Base
  has_attached_file :cover, :styles => {:small => 'x100', :large => '1000x1000>'}
  has_attached_file :sample, :styles => {:thumb => 'x100'}
end
```

Then in `RAILS_ROOT/public/system/paperclip_attachments.yml`:
然后在`RAILS_ROOT/public/system/paperclip_attachments.yml`：

```yml
---
:User:
  :avatar:
  - :thumb
  - :croppable
  - :big
:Book:
  :cover:
  - :small
  - :large
  :sample:
  - :thumb
```

---

Testing  测试
-------

Paperclip provides rspec-compatible matchers for testing attachments.
Paperclip 提供了一个兼容rspec的匹配器来测试附件。
See the documentation on [Paperclip::Shoulda::Matchers](http://rubydoc.info/gems/paperclip/Paperclip/Shoulda/Matchers) for more information.
更多信息参见文档。

**Parallel Tests**  并行测试

Because of the default `path` for Paperclip storage, if you try to run tests in parallel, you may find that files get overwritten because the same path is being calculated for them in each test process.
因为Paperclip存储默认`path`的原因，如果尝试并行运行测试，会发现文件总被改写，因为在每次测试过程中会有相同的路径被计算出来。
While this fix works for parallel_tests, a similar concept should be used for any other mechanism for running tests concurrently.
为了修复并行测试，一个相似的概念要运用于其他机制。

```ruby
if ENV['PARALLEL_TEST_GROUPS']
  Paperclip::Attachment.default_options[:path] = ":rails_root/public/system/:rails_env/#{ENV['TEST_ENV_NUMBER'].to_i}/:class/:attachment/:id_partition/:filename"
else
  Paperclip::Attachment.default_options[:path] = ":rails_root/public/system/:rails_env/:class/:attachment/:id_partition/:filename"
end
```

The important part here being the inclusion of `ENV['TEST_ENV_NUMBER']`, or the similar mechanism for whichever parallel testing library you use.
这里重要的是包含了`ENV['TEST_ENV_NUMBER']`，或者使用其他并行测试库时类似的机制

**Integration Tests**  集成测试

Using integration tests with FactoryGirl may save multiple copies of your test files within the app.
使用FactoryGirl进行集成测试可能会将测试文件保存成多个副本。
To avoid this, specify a custom path in the `config/environments/test.rb` like so:
要避免这个问题，需要在`config/environments/test.rb`指定一个自定义路径，比如：

```ruby
Paperclip::Attachment.default_options[:path] = "#{Rails.root}/spec/test_files/:class/:id_partition/:style.:extension"
```

Then, make sure to delete that directory after the test suite runs by adding this to `spec_helper.rb`.
然后，通过在`spec_helper.rb`文件添加下面这行，确保在测试套件运行之后删除那个目录。

```ruby
config.after(:suite) do
  FileUtils.rm_rf(Dir["#{Rails.root}/spec/test_files/"])
end
```
---

Contributing  贡献
------------

If you'd like to contribute a feature or bugfix: Thanks!
To make sure your fix/feature has a high chance of being included, please read the following guidelines:


1. Post a [pull request](https://github.com/thoughtbot/paperclip/compare/).
2. Make sure there are tests! We will not accept any patch that is not tested.
   It's a rare time when explicit tests aren't needed. If you have questions
   about writing tests for paperclip, please open a
   [GitHub issue](https://github.com/thoughtbot/paperclip/issues/new).

Please see `CONTRIBUTING.md` for more details on contributing and running test.

Thank you to all [the contributors](https://github.com/thoughtbot/paperclip/contributors)!

License  许可证
-------

Paperclip is Copyright © 2008-2015 thoughtbot, inc. It is free software, and may be
redistributed under the terms specified in the MIT-LICENSE file.

About thoughtbot  关于作者
----------------

![thoughtbot](https://thoughtbot.com/logo.png)

Paperclip is maintained and funded by thoughtbot.
The names and logos for thoughtbot are trademarks of thoughtbot, inc.

We love open source software!
See [our other projects][community] or
[hire us][hire] to design, develop, and grow your product.

[community]: https://thoughtbot.com/community?utm_source=github
[hire]: https://thoughtbot.com?utm_source=github
# vim: set filetype=markdown : 
