# 第七章 注册

既然我们有了User模型，是时候添加一些几乎所有网站都有的功能：用户注册。我们使用HTML表单来提交用户注册信息（7.2节），然后通过它来创建新用户，最后把用户信息保存至数据库（7.4节）。在注册过程的最后，还有一件重要的事就是渲染新创建的用户信息页面。所以我们先通过创建用户显示页面开始，它为用户的REST架构（2.2.2节）实现迈出第一步。沿着这个方向，我们将在5.3.4节的基础上再写一些简洁但是富有表达力的集成测试。

本章我们会依赖用户模型的有效性验证增加新用户email地址有效的概率。在第十章，我们会通过添加账户激活的方法来再次确保注册用户提交email是有效的email地址。

## 7.1 显示用户

本节我们先创建一个显示用户名和头像的网页，这是构建用户信息页面的第一步，如图7.1里原型图表示的。用户信息页面包含用户个人信息和该用户发送的微博列表，如图7.2的草图。（图7.2有我们的第一个lorem ipsum文本的例子，关于lorem ipsum有个迷人的故事，你有时间一定要了解一下。）我们将在第十二章完成这个任务，同时我们的Sample App也就构建完成。

假如你一直在跟随本教程使用版本控制，那么如往常一样再次创建一个主题分支：

```
$ git checkout master
$ git checkout - b sign-up
```

![图7.1：本节中用户信息页面草图](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/profile_mockup_profile_name_bootstrap.png)

![图7.2：完整的用户信息页面草图](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/profile_mockup_bootstrap.png)

### 7.1.1 调试和Rails环境

本节的用户信息页面是我们的应用程序中第一个真正的动态网页。尽管视图只有一个，但是每个用户信息页面都是先通过使用从数据库中获取的数据填充视图，然后应用程序把填充后的页面返回给浏览器。
为了为我们的Sample App添加动态页面做好准备，先把调试信息添加到我们网站的布局文件里（代码清单7.1）。

```ruby
代码清单7.1：为网站布局文件添加调试信息。
# app/views/layouts/application.html.erb
<!DOCTYPE html>
<html>
  .
  .
  .
  <body>
    <%= render 'layouts/header' %>
    <div class="container">
      <%= yield %>
      <%= render 'layouts/footer' %>
      <%= debug(params) if Rails.env.development? %>
    </div>
  </body>
</html>
```

因为我们不想在已经部署的应用里显示调试信息，代码清单7.1使用

```ruby
if Rails.env.development?
```
来限制调试信息只是在开发环境时才输出至网页，开发环境（development）是Rails默认的三种环境之一（注7.1）。具体来说，仅仅在开发环境中Rails.env.development?值才是true，所以内嵌Ruby
```ruby
<%= debug(params) if Rails.env.development? %>
```
不会在生产环境或测试环境下输出调试信息。（把调试信息插入测试可能不会有任何坏处，但是它也不会起到任何作用，所以最好是限制调试信息仅仅在开发环境时才显示。）

    注7.1 Rails环境
    Rails自带三个环境：测试（test）、开发（development）和生产（production）。默认的Rails控制台命令是开发环境：

    '''ruby
    $ rails console
    Loading development environment
    >> Rails.env
    => "development"
    >> Rails.env.development?
    => true
    >> Rails.env.test?
    => false
    ```
    正如你所见，Rails提供了Rails对象，有一个env属性和相联系的逻辑方法的环境，以便，例如，Rails.env.test?返回true在测试环境里，否则false。

    假如你需要在不同的环境运行控制台（为了调试测试，例如），你可以把环境作为参数传递给控制台：
    ```ruby
    $ rails console test
    Loading test environment
    >> Rails.env
    => "test"
    >> Rails.env.test?
    => true
    ```
    正如用控制台，开发环境是默认的Rails server的环境，但是你也可以在不同的环境运行。

    $ rails server --environment production

    假如你在生产环境里浏览你的app，没有生产数据库它不会运行，我们可以通过在生产环境运行rake db:migrate：

    $ bundle exec rake db:migrate RAILS_ENV=production

    (我发现控制台、服务器和迁移命令在三个可变的不兼容的方法里没有默认的环境，这是为什么我显示了所有撒个的原因）

    顺便提一下，假如你已经部署你的应用到Heroku，你能使用heroku run console来看它的环境：

    $ heroku run console
    >> Rails.env
    => "production"
    >> Rails.env.production?
    => true

    自然，因为Heroku是产品网站的平台，它在生产环境运行每个应用程序。


我们在第五章里的自定义样式表里再添加一些样式，美化一下输出的调试信息。如代码清单7.2所示：

代码清单7.2： 添加美化调试信息的代码，包括一个Sass的mixin。
```sass
# app/assets/stylesheets/custom.css.scss
@import "bootstrap-sprockets";
@import "bootstrap";

/* mixins, variables, etc. */

$gray-medium-light: #eaeaea;

@mixin box_sizing {
  -moz-box-sizing:    border-box;
  -webkit-box-sizing: border-box;
  box-sizing:         border-box;
}
.
.
.
/* miscellaneous */

.debug_dump {
  clear: both;
  float: left;
  width: 100%;
  margin-top: 45px;
  @include box_sizing;
}
```
这里引入了Sass mixin工具，在这里叫box_sizing。mixin允许打包一组CSS，然后在为其他标签定义CSS规则时重复使用。

```sass
.debug_dump {
  .
  .
  .
  @include box_sizing;
}
```
转换为CSS的代码为

```css
.debug_dump {
  .
  .
  .
  -moz-box-sizing:    border-box;
  -webkit-box-sizing: border-box;
  box-sizing:         border-box;
}
```
我们在7.2.1节会再次使用这个mixin。这个例子中，调试信息如图7.3所示。

![图7.3：带调试信息的示例程序主页](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/home_page_with_debug_3rd_edition.png)

在图7.3里给出了被渲染的页面的一些有用的调试信息：

```yaml
---
controller: static_pages
action: home
```
这是YAML的表示方法，它和哈希差不多。在这个例子里给出的信息是控制器的名称和执行的动作。我们在7.1.2节里会看到其他例子。

### 7.1.2 用户资源

为了创建用户信息页面，需要在数据库中有至少一个用户。这产生了一个鸡生蛋、蛋生鸡的问题：网站怎么能做到在还没有注册页面之前就为数据库中添加用户？高兴的是，这个问题已经解决了：在6.3.4节，我们使用Rails控制台手动创建一条User记录，所以现在数据库中有一个用户：

```ruby
$ rails console
>> User.count
=> 1
>> User.first
=> #<User id: 1, name: "Michael Hartl", email: "mhartl@example.com",
created_at: "2014-08-29 2：8：28", updated_at: "2014-08-29 2：8：28",
password_digest: "$2a$10$YmQTuuDNOszvu5yi7auOC.F4G//FGhyQSWCpghqRWQW...">
```

（假如你的数据库里目前还没有用户，现在你应该再看看6.3.4节，然后在进行下一步前先在数据库中添加一个用户）我们从控制台的输出了解到用户id为1，我们现在的目的是创建显示这个用户信息的页面。我们将遵循REST架构的惯例（注2.2），也就是说HTTP标准定义（注3.2）的数据库中的数据通过浏览器被创建（create）、浏览（show）、更新（update）或删除（Destroy）的四个动作对应的四个操作方式POST、GET、PATCH和DELETE。

当遵循REST原理时，资源（resource）典型的用法就是我们通过资源的名称或id来查找该资源。意思是在用户的环境--我们现在正考虑User资源-对于id等于1的用户，我们通过GET请求URL地址 /users/1来浏览该用户的信息。这里show动作暗示这种类型动作--当rails的REST特性被激活时，GET请求被show动作自动处理了。

我们在2.2.1节看到id为1的用户对应的URL地址为 /users/1。
不幸的是，现在访问这个URL只会抛出一个错误信息，如同我们在服务器日志里看到的一样（图7.4）

![图7.4：为/users/1的服务器日志](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/profile_routing_error_3rd_edition.png)

我们能通过在我们的路由文件中（config/routes.rb）添加一行代码来让/users/1工作。

```ruby
resources :users
```
结果如代码清单7.3所示。

```ruby
代码清单7.3：把User资源添加到路由文件。
# config/routes.rb
Rails.application.routes.draw do
  root             'static_pages#home'
  get 'help'    => 'static_pages#help'
  get 'about'   => 'static_pages#about'
  get 'contact' => 'static_pages#contact'
  get 'signup'  => 'users#new'
  resources :users
end
```
尽管我们的现在的相反是创建一个显示用户信息的页面，但是这行代码`resources :users`不是单单地添加了users/1这样的URL;
它使我们的应用程序一下拥有了所有我们需要的REST的Users资源，和许多具名路由一起（5.3.3节）为用户资源生成了相应的URL。相应的URL以及动作和具名路由显示在表7.1。（和表2.2比较）接下来三章，我们将覆盖表7.1里面的所有内容。当我们补充完所有必要的动作时，User就会成为一个完整的REST的资源。


HTTP请求| URL | 动作｜目的
——--|---|---|---
GET | /users | index| users_path
GET | /users/1 |show| user_path(user)
GET | /users/new|create|new_user_path
POST|/users |create|users_path
GET|/users/1/edit|edit/edit_user_path(user)
PATCH|/users/1|update user_path(user)
DELETE|/users/1|destroy|user_path(user)

表７．１：在代码清单7.3里User资源提供的的路由

有了代码清单7.3里的代码，路由可以正常工作了，但是现在依然没有相应的页面（图7.5）。为了解决这个，我们将开始创建一个迷你版的用户信息页面，然后在7.1.4节进一步充实。

![图7.5：URL地址/users/1有路由没有页面](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_show_unknown_action_3rd_edition.png)

我们将使用标准的Rails位置来显示用户，就是app/views/users/show.html.erb。不像new.html.erb视图，我们在代码清单5.28里用生成器生成它，show.html.erb文件目前不存在，所以你不得不手动创建它，然后填入代码清单7.4里的代码。

```ruby
代码清单7.4：显示用户信息的占位视图。
# app/views/users/show.html.erb
<%= @user.name %>, <%= @user.email %>
```

设想有一个名为@user的实例变量，这个视图使用了内嵌的Ruby代码来显示用户的姓名和email地址。当然，最后真实的用户显示页面和这个肯定差别很大（也不会公开显示email地址）。

为了让显示用户的视图工作，我们需要在相应的Users控制器里的定义一个show动作。你可能已经想到，我们通过在User模型上使用find方法（6.1.4节）从数据库中取出用户，如代码清单7.5所示。

```ruby
代码清单7.5：带show动作的User控制器。
# app/controllers/users_controller.rb
class UsersController < ApplicationController

  def show
    @user = User.find(params[:id])
  end

  def new
  end
end
```

这里我们使用params来取回用户id，params[:id]的值为1，意思是是用户id为1。所以这里的代码和我们在6.1.4节见过的find方法User.find(1)的效果是一样的。（严格的说，params[:id]是字符串“1”，但是find是足够聪明的，会自动把它转化为整数。）

随着用户视图和动作被我们定义，URL地址/users/1现在可以完美的工作了，如图7.6所见。（假如你自从添加了bcrypt以后还没有重启Rails服务器，你这次可能不得不重启一下服务器了。）注意图7.6的调试信息，确认params[:id]的值为：
```yaml
---
action: show
controller: users
id: '1'
```

这就是为什么代码
```ruby
User.find(params[:id])
```
会找到id为1的用户。

![图7.6：添加用户资源后的用户显示页面](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_show_3rd_edition.png)

### 7.1.3 调试

我们在7.1.2节里见识过了调试信息可以怎样帮助我们理解程序里发生了些什么。对于Rails 5来说，有一个更直接的方法是使用byebug gem(代码清单3.2）进行调试。为了了解一下它究竟是怎么工作的，我们只需要在我们的程序中添加加一行代码---`debugger`，正如代码清单7.6所示。

```ruby
代码清单7.6： 带调试器的用户控制器。
# app/controllers/users_controller.rb
class UsersController < ApplicationController

  def show
    @user = User.find(params[:id])
    debugger
  end

  def new
  end
end
```

现在，当我们访问 /users/1时, Rails服务器将显示一个byebug提示：
```ruby
(byebug)
```

我们可以像Rails控制台一样对待它，通过命令来查明应用程序的状态：
```ruby
(byebug) @user.name
"Example User"
(byebug) @user.email
"example@railstutorial.org"
(byebug) params[:id]
"1"
```
要释放调试继续执行程序，按Ctrl-D，然后从show动作中移除debugger（代码清单7.7）。

```ruby
代码清单7.7：移除debugger后的User控制器。
# app/controllers/users_controller.rb
class UsersController < ApplicationController

  def show
    @user = User.find(params[:id])
  end

  def new
  end
end

```

你在Rails程序中无论碰到什么困惑，把debugger放在你认为会引起问题的代码附近是一个好编程的实践。使用byebug侦查系统的状态是追踪应用程序的错误和交互调试应用程序是有利的方法。

### 7.1.4 Gravatar 图片和侧边栏

在先前的一节里已经定义了一个基本的用户页面，我们现在为每个用户添加头像和第一版的用户侧边栏来充实用户信息页面。我们通过添加一个“全球可认识的头像”，或者![Gravatar](http://gravatar.com/)到用户信息页面。Gravata是一个允许用户上传图片然后用email地址把它们俩联系到一起的免费服务。因此，Gravatar是不需要经历图片上传、切割和储存的方便方法；我们需要做得是使用用户的email地址构建正确的Gravatar图片URL，相应的Gravatar图片就会自动显示。（我们将在11.4节中学习怎样处理自定义上传图片）

我们的计划是定义一个gravtar_for辅助方法来为给定的用户返回对应的Gravatar图片，如代码清单7.8所示。

```ruby
代码清单7.8： 带头像和用户姓名的show视图。
# app/views/users/show.html.erb

<% privide(:title, @user.name) %>
<h1>
  <%= gravatar_for @user %>
  <%= @user.name %>
</h1>
```

默认地，定义在任何辅助文件里的方法会自动的在所有视图都可用，但是为了方便，我们将把gravatar_for辅助方法方法放在和和Users控制器关联的文件里。如在[Gravatar文档](http://en.gravatar.com/site/implement/hash/)里提到的，Gravatar URL是以用户的email地址为基础的MD5哈希。在Ruby里，MD5哈希算法使用hexdigest方法实现，它是Digest库的一部分：

```ruby
>> email = "MHARTL@example.COM"
>> Digest::MD5::hexdigest(email.downcase)
=> "1fda4469bcbec3badf5418269ffc5968"
```

因为email地址是忽略大小写的（6.2.4节）但是MD5哈希不是，所以我们将使用downcase方法来确保hexdigest方法的参数都是小写。（因为在代码清单6.31里的email小写回叫函数，在本书里gravatar_for方法中email是否调用downcase方法不会有任何不同，但是万一gravatar_for从其他不同的地方取得了email地址，就会不一样了。）因此gravatar_for辅助方法的代码如代码清单7.9所示。
```ruby
代码清单7.9：定义gravatar_for 辅助方法。
# app/helpers/users_helper.rb
module UsersHelper

  # Returns the Gravatar for the given user.
  def gravatar_for(user)
    gravatar_id = Digest::MD5::hexdigest(user.email.downcase)
    gravatar_url = "https://secure.gravatar.com/avatar/#{gravatar_id}"
    image_tag(gravatar_url, alt: user.name, class: "gravatar")
  end
end
```
在代码清单7.9里的代码返回image_tag，定义CSS类为“gravatar”，添加了以用户名字作为这个图片标签的alt文本（它对视觉有障碍的用户使用屏幕阅读器是尤其方便的）。

简介页面如图7.7所示，它显示默认的Gravatar图片，这是因为user@example.com不是一个真实的email地址。

![图7.7：带默认Gravatar的默认用户信息页面](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/profile_with_gravatar_3rd_edition.png)

为了让我们的程序显示自定义Gravatar，我们将使用update_attributes(6.1.5节）来把用户的email改为我控制的email账户：

```ruby
$ rails console
>> user = User.first
>> user.update_attributes(name: "Example User",
?>                        email: "example@railstutorial.org",
?>                        password: "foobar",
?>                        password_confirmation: "foobar")
=> true
```
这里是我们分配email地址example@railstutorial.org给我们的用户，它已经由我和Rails Tutorial标志联系起来，如图7.8所见。

![图7.8：自定义Gravatar的用户显示页面](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/profile_custom_gravatar_3rd_edition.png)

要完成图7.1的草图最后剩余的一点工作是添加初始版本的侧边栏。

我们使用aside标签来实现它，aside标签常常用来弥补剩余的页面，但是也能单独使用。我们引入row和col-md-4两个CSS的样式类，它们都是Bootstrap的一部分。修改后的用户信息显示视图代码如代码清单7.10：
```ruby
代码清单7.10： 为用户信息显示视图添加侧边栏（sidebar）
# app/views/users/show.html.erb
<% provide(:title, @user.name) %>
<div class="row">
  <aside class="col-md-4">
    <section class="user_info">
      <h1>
        <%= gravatar_for @user %>
        <%= @user.name %>
      </h1>
    </section>
  </aside>
</div>
```
添加了HTML和CSS类后，我们可以为用户信息页面（包含侧边栏和Gravatar）添加一些样式，如代码清单7.11显示的SCSS代码。（注意asset pipeline使用Sass引擎将代码编译为CSS），结果页面如图7.9所示。

```scss
代码清单7.11：为带侧边栏的用户信息页面添加样式。
# app/assets/stylesheets/custom.css.scss
 .
.
.
/* sidebar */

aside {
  section.user_info {
    margin-top: 20px;
  }
  section {
    padding: 10px 0;
    margin-top: 20px;
    &:first-child {
      border: 0;
      padding-top: 0;
    }
    span {
      display: block;
      margin-bottom: 3px;
      line-height: 1;
    }
    h1 {
      font-size: 1.4em;
      text-align: left;
      letter-spacing: -1px;
      margin-bottom: 3px;
      margin-top: 0px;
    }
  }
}

.gravatar {
  float: left;
  margin-right: 10px;
}

.gravatar_edit {
  margin-top: 15px;
}
```

![图7.9：侧边栏和CSS显示的用户显示页面](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_show_sidebar_css_3rd_edition.png)

## 7.2 注册表单

既然我们已经有了用户信息页面（还没完成），我们准备为我们的网站创建一个注册表单。我们在图5.9里见过（在图7.10里再次显示）的注册页面是空的：除非是注册新用户。这节的目标是开始通过创建图7.11里的草图所描述的页面开始改变这个悲催的事情。

![图7.10：现在的注册页面状态/signup](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/new_signup_page_3rd_edition.png)

![图7.11：用户注册页面的草图](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/signup_mockup_bootstrap.png)

因为我们准备为网站添加创建新用户的能力，所以让我们先移除在6.3.4节中通过控制台创建的用户。最彻底的方法是重置数据库，用db:migrate:reset这个Rake任务：

```terminal
$bundle exec rails db:migrate:reset
```

在有些操作系统，你可能不得不重启网站服务器（使用Ctrl-C）结束服务器，然后重新启动服务器。

### 7.2.1 使用form_for
注册页面的核心是用来提交相关注册信息（name，email，password和password_confirmation）的表单。在Rails中我们使用form_for辅助方法来完成表单，它以一个ApplicationRecord对象为参数，然后使用这个对象的属性构建表单。

还记得注册页面的URL地址/signup被路由到User控制器的new动作（代码清单5.33）吗？我们要做的第一步是创建一个User对象，然后把它作为form_for的参数，如代码清单7.12所示：
```ruby
代码清单7.12：为动作new添加@user变量。
# app/controllers/users_controller.rb
class UsersController < ApplicationController

  def show
    @user = User.find(params[:id])
  end

  def new
    @user = User.new
  end
end
```
表单如代码清单7.13所示。我们将在7.2.2节详细的讨论它，但是先用代码清单7.14里的SCSS代码来为表单添加一点样式。（注意在代码清单7.2中box_sizing 的重用）一旦应用了这些CSS规则，注册页面如图7.12所示：
```erb
代码清单7.13：新用户注册的表单。
# app/views/users/new.html.erb
<% provide(:title, 'Sign up') %>
<h1>Sign up</h1>

<div class="row">
  <div class="col-md-6 col-md-offset-3">
    <%= form_for(@user) do |f| %>
      <%= f.label :name %>
      <%= f.text_field :name %>

      <%= f.label :email %>
      <%= f.email_field :email %>

      <%= f.label :password %>
      <%= f.password_field :password %>

      <%= f.label :password_confirmation, "Confirmation" %>
      <%= f.password_field :password_confirmation %>

      <%= f.submit "Create my account", class: "btn btn-primary" %>
    <% end %>
  </div>
</div>
```

```scss
代码清单7.14：注册表单的CSS。
# app/assets/stylesheets/custom.css.scss
 .
.
.
/* forms */

input, textarea, select, .uneditable-input {
  border: 1px solid #bbb;
  width: 100%;
  margin-bottom: 15px;
  @include box_sizing;
}

input {
  height: auto !important;
}
```
![图7.12：用户注册表格](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/signup_form_3rd_edition.png)

### 7.2.2 注册表单HTML
为了理解在代码清单7.13里定义的表格，我们把它们分解开来是有助于理解的。我们先看看外部结构，它包含了内嵌Ruby代码、用form_for打开、用end关闭：

```ruby
<%= form_for(@user) do |f| %>
  .
  .
  .
<% end %>
```
do关键词的出现显示form_for用带一个变量的块作为参数，我们叫它f（代表“form”）。

和Rails辅助方法的通常做法一样，我们不需要知道内部实现的任何细节，但是我们确实需要知道f对象做什么:
当f带一个与HTML form标签相对应的方法被调用时--例如文本框、单选按钮或者密码框--f为这个标签返回用来设置@user对象属性的代码。换句话说，
```erb
<%= f.label :name %>
<%= f.text_field :name %>
```
为@user的name属性生成了HTML标签和文本框。

你可以通过在浏览器里Ctr-点击然后选择“查看元素”的功能来看看生成的HTML。注册页面的网页源代码看上去应该如代码清单7.15所示。让我们花几分钟来讨论一下它的结构。
```html
代码清单7.15：在图7.12里表单对应的HTML代码。
<form accept-charset="UTF-8" action="/users" class="new_user"
      id="new_user" method="post">
  <input name="utf8" type="hidden" value="&#x2713;" />
  <input name="authenticity_token" type="hidden"
         value="NNb6+J/j46LcrgYUC60wQ2titMuJQ5lLqyAbnbAUkdo=" />
  <label for="user_name">Name</label>
  <input id="user_name" name="user[name]" type="text" />

  <label for="user_email">Email</label>
  <input id="user_email" name="user[email]" type="email" />

  <label for="user_password">Password</label>
  <input id="user_password" name="user[password]"
         type="password" />

  <label for="user_password_confirmation">Confirmation</label>
  <input id="user_password_confirmation"
         name="user[password_confirmation]" type="password" />

  <input class="btn btn-primary" name="commit" type="submit"
         value="Create my account" />
</form>
```

我们从文档的内部结构开始。比较代码清单7.13和代码清单7.15，我们看见内嵌Ruby
```erb
<%= f.label :name %>
<%= f.text_field :name %>
```
产生的HTML代码为
```html
<label for="user_name">Name</label>
<input id="user_name" name="user[name]" type="text" />
```
然而
```ruby
<%= f.label :email %>
<%= f.email_field :email %>
```
产生的HTML
```html
<label for="user_email">Email</label>
<input id="user_email" name="user[email]" type="email" />
```
和
```ruby
<%= f.label :password %>
<%= f.password_field :password %>

```
产生的HTML
```html
<label for="user_password">Password</label>
<input id="user_password" name="user[password]" type="password" />
```
如在图7.13所见，文本和email框（type="text" 和type="email"）直接显示用户输入，然而密码框（type="password")出于安全目的隐藏了用户输入，如
图7.13所示。（使用email框的好处是有些系统对待它和对待文本框不一样：例如，代码type="email"将引起一些移动设备显示特殊优化过的键盘，方便输入email地址。）


![图7.13：带text和password文本框的填充后的表格](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/filled_in_form_bootstrap_3rd_edition.png)

如我们在7.4节所见，创建用户的关键是每个input特殊的name属性：
```html
<input id="user_name" name="user[name]" - - - />
.
.
.
<input id="user_password" name="user[password]" - - - />
```

这些name值允许Rails构建一个初始化的哈希（通过params变量）来根据用户输入的值来创建用户，如7.3节所见。

第二个需要理解的重点是form标签本身。Rails通过@user对象创建form标签：因为每个Ruby对象都知道它本身归属的是那个类（4.4.1节），Rails知道@user属于User类，而且因为@user是一个新用户，Rails也知道需要采用post方法来创建表单，POST是创建新对象正确的动词（注3.2）：
```html
<form action="/users" class="new_user" id="new_user" method="post">
```
这里class和id属性只是为了方便我们设置CSS规则，重要的是action="/users" 和method="post"。它们的意思是发出一个HTTP POST请求到/users
URL。我们会在下两节看到它的作用。

（你可能也注意到显示在form标签里的其他代码：
```html
 <div style="display:none">
    <input name="utf8" type="hidden" value="&#x2713;" />
    <input name="authenticity_token" type="hidden"
           value="NNb6+J/j46LcrgYUC60wQ2titMuJQ5lLqyAbnbAUkdo=" />
  </div>
```
这段代码并没有在浏览器里显示，Rails需要用它来提高网站的安全性，所以它对我们理解form_for没有帮助。简短地说，它使用Unicode字符&#x2713;(对号）来强迫浏览器使用正确的字符编码来提交数据，另外它还包含一个授权口令，Rails用它来防止跨网站请求劫持（CSRF）。

### 7.3 不成功的注册

尽管我们已经简单地检验了一下图7.12（显示在代码清单7.15里）HTML，但是我们还没有覆盖任何细节。失败的注册有助于我们理解表单，因此本节我们先创建一个注册表单，然后提交一些无效的注册信息，Rails会刷新注册页面，并且显示一列相关的错误信息，如图7.14显示的原型。

![图7.14：注册失败页面草图](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/signup_failure_mockup_bootstrap.png)

### 7.3.1 可用表单
回忆一下在7.12节，我们把`resources :users`添加到routes.rb文件中（代码清单7.3），它会确保我们的Rails应用程序自动响应表7.1描述的REST的URL。具体来说，它确保POST到/users的请求会被create动作处理。我们为create动作设置的策略是通过提交表单使用User.new创建了一个新的user对象，试试（并且失败的)保存用户，然后为了重新提交的可能性来渲染注册页面。让我们通过审查注册表单的代码开始：
```html
<form action="/users" class="new_user" id="new_user" method="post">
```

如7.2.2节提过的，这段HTML通过POST方法发送请求到URL地址/users。

我们朝一个可用的注册表单行进的第一步是添加代码清单7.16里的代码。这也引出了render方法的另一种用法，我们在5.1.3节的Partial中看见过的；正如我们所见，render在控制器的动作里也可以工作。我们抓住这个机会来介绍一下if-else分支结构，它允许我们依据@user.save的值处理失败和成功的情况，（正如我们在6.1.3所见)而它的值是true或false依赖于@user是否保存成功。

```ruby
代码清单7.6：可以处理注册失败的create动作。
# app/controllers/users_controller.rb
class UsersController < ApplicationController

  def show
    @user = User.find(params[:id])
  end

  def new
    @user = User.new
  end

  def create
    @user = User.new(params[:user])    # 不是最终的实现！
    if @user.save
      # 处理成功保存情形
    else
      render 'new'
    end
  end
end
```
注意注释：这不是最后的实现。但是它足够我们开始了，我们将在7.3.2节里完成它。

理解代码清单7.16里的代码的最好方法是为表单提交无效注册数据。结果如图7.15显示，完整的调试信息显示在图7.16里。（图7.15也显示了网页控制台（web console），它会在浏览器里打开Rails控制台来辅助调试。它对实验是有用的，例如，用户模型，但是现在，我们需要做的是查看params变量，到目前为止在web控制台里还做不到。）

![图7.15：注册失败](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/signup_failure_3rd_edition.png)
![图7.16：注册失败调试信息](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/signup_failure_debug_3rd_edition.png)

为了对Rails怎样处理提交有个较好的理解，让我走近看看user的部分哈希参数，从调试信息（图7.16）：

```ruby
"user" => { "name" => "Foo Bar",
            "email" => "foo@invalid",
            "password" => "[FILTERED]",
            "password_confirmation" => "[FILTERED]"
          }
```
这个哈希作为params的部分参数被传递给用户控制器。我们看到从7.1.2节开始，params哈希包含每个请求的相关信息。在像/users/1这样的URL里，params[:id]就是用户相应的id（在这里是1）。在post注册表格，params是包含一个哈希的哈希，我们在4.3.3节学习过的数据结构。我们在控制台的会话里针对性的命名为变量params。上面的调试信息显示提交表单的信息被保存到名为user的哈希中，这个哈希对应的键值和表单相应的属性对应，键就是input标签的name属性，而值就是我们在对应的属性上输入的内容。代码清单7.13中有，例如，
```html
<input id="user_email" name="user[email]" type="email" />
```
“user[email]”也恰好是user哈希的email属性。

尽管哈希键在调试输出里显示的是字符串，但是我们可以在User控制器里把它们当作symbol，所以params[:user]就是用户属性的哈希--事实上，它正好就是User.new需要的参数，如在4.4.5节里第一次见过的然后在代码清单7.16里又看到的一样。这意味着这行
```ruby
@user = User.new(params[:user])
```
代码几乎和
```ruby
@user = User.new(name: "Foo Bar", email: "foo@invalid",
                 password: "foo", password_confirmation: "bar")
```
是等价的。在之前的Rails版本，使用
```ruby
@user = User.new(params[:user])
```
就会工作了，但是因为它默认不安全，所以需要小心使用，而且在防止恶意用户偷偷地修改应用程序数据库时容易出错。在Rails4.0以后，这段代码会抛出错误（如图7.15和7.16所见），意思是它是默认安全的。

### 7.3.2 “健壮参数”（Strong Parameter）

我们在4.4.5节简单地提到了大量赋值，需要使用哈希值来初始化Ruby变量，如下面代码
```ruby
@user = User.new(params[:user]) #不是最后的实现
```

在代码清单7.16里的注释和上面的重现表明这不是最后的实现。原因是初始化整个params哈希是尤其危险的--它把用户提交的所有数据都传送给User.new。具体来说，除当前的属性外，假设User模型包含admin属性，常常用来识别网站的管理员的属性。（我们在9.4.1节实现这个属性）设置这个属性为true的方法是传递admin='1'作为params[:user]的一部分。完成这个任务是很容易的，使用命令行HTTP客户端软件例如curl都可以完成。结果是通过传递整个params哈希给User.new，我们允许网站的任意用户取得管理员权限，只要参数params中包含admin='1'这样的值。

Rails之前的版本在模型层使用了attr_accessible的方法解决这个问题，你仍然在以前的Rails应用程序里看到这个方法，但是Rails 4.0以后更喜欢的技术是在控制器使用叫做“健壮参数”（Strong Parameter）的技术。这允许我们特别指明那个参数被请求，那个参数被许可。另外，用上面的方法传递原始的params哈希会抛出错误，所以现在的Rails应用默认情况下对大量赋值漏洞的已经免疫。

目前的情形是，我们想要params哈希有:user属性，许可name、email、password和password_confirmation属性（没有其他的）。我们通过下面完成：

```ruby
params.require(:user).permit(:name,:email,:password,:password_confirmation)
```
这段代码返回仅包含被许可的属性的哈希版本（当:user属性丢失会抛出错误）。

为了便于使用这些参数，我们通过引入user_params辅助方法（返回合适的初始化哈希），用它代替params[:user]:

```ruby
@user = User.new(user_params)
```

因为user_params只在User控制器的内部使用，不需要通过网页来暴露到外部，我们使用Ruby的private关键词来把它变成私有的，如代码清单7.17所示。（我们将在8.4节讨论更多关于private的内容）

```ruby
代码清单7.7：在create动作里使用“健壮参数”（Strong Parameter）。
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  .
  .
  .
  def create
    @user = User.new(user_params)
    if @user.save
      # Handle a successful save.
    else
      render 'new'
    end
  end

  private

    def user_params
      params.require(:user).permit(:name, :email, :password,
                                   :password_confirmation)
    end
end
```

顺便提一下，user_params额外的缩进是设计来显示那些方法是在private之后定义的。（经验显示这是明智的，在有大量方法的类里，不小心定义私有方法很可能，当它在相应的对象里不可用时会引起我们困惑）

现在注册表格可用了，起码在某种意义上，因为提交时没有错误了。换句话说，如图7.17所见，它对无效的提交不显示任何反馈（除开发环境的调试区域外），这有点让人困惑。它也实际上没有创建新用户。我们将在7.3.3节解决第一个问题，在7.4节解决第二个。

![图7.17：提交了无效信息后的注册页面](http://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/invalid_submission_no_feedback.png)

### 7.3.3 注册错误页面
作为处理创建用户失败的最后一步，我们将添加有用的错误信息来说明阻止成功注册的原因。为了方便，Rails依据User模型的验证信息自动提供了这类的信息。例如，考虑一下用无效的email地址和很短的密码来注册用户：

```ruby
$ rails console
>> user = User.new(name: "Foo Bar", email: "foo@invalid",
?>                 password: "dude", password_confirmation: "dude")
>> user.save
=> false
>> user.errors.full_messages
=> ["Email is invalid", "Password is too short (minimum is 6 characters)"]
```
这里errors.full_messages对象（在6.2.2节提过）包含一个错误信息数组。

和上面的控制台会话一样，在代码清单7.16里保存失败生成了一列和@user对象相关的错误。为了在浏览器里显示这些信息，我们将在用户new页面渲染错误信息Partial，同时添加为所有的输入框添加CSS类form-control(对Bootstrap有特殊的意义），如代码清单7.18所示。值得一提的是错误信息片段（Partial）只是我们的首次尝试，最后的版本如11.3.2显示。
```ruby
代码清单7.8：在注册表单显示错误信息的代码。
# app/views/users/new.html.erb
<% provide(:title, 'Sign up') %>
<h1>Sign up</h1>

<div class="row">
  <div class="col-md-6 col-md-offset-3">
    <%= form_for(@user) do |f| %>
      <%= render 'sha红色/error_messages' %>

      <%= f.label :name %>
      <%= f.text_field :name, class: 'form-control' %>

      <%= f.label :email %>
      <%= f.email_field :email, class: 'form-control' %>

      <%= f.label :password %>
      <%= f.password_field :password, class: 'form-control' %>

      <%= f.label :password_confirmation, "Confirmation" %>
      <%= f.password_field :password_confirmation, class: 'form-control' %>

      <%= f.submit "Create my account", class: "btn btn-primary" %>
    <% end %>
  </div>
</div>
```
这里注意我们render了一个名为'sha红色/error_messages'的视图片段。sha红色目录专为想要在多个控制器里重复使用的视图片段设计。（我们将在9.1.1里看见完整的解释）这意味着我们不得不使用mkdir创建新的app/views/sha红色目录（表1.1）：

```terminal
$ mkdir # app/views/sha红色
```
然后我们需要创建_error_messages.html.erb视图片段文件，使用我们和平常一样的文本编辑器。视图片段的内容如代码清单7.19：
```ruby
代码清单7.9：显示表单提交错误信息的视图片段。
# app/views/sha红色/_error_messages.html.erb
<% if @user.errors.any? %>
  <div id="error_explanation">
    <div class="alert alert-danger">
      The form contains <%= pluralize(@user.errors.count, "error") %>.
    </div>
    <ul>
    <% @user.errors.full_messages.each do |msg| %>
      <li><%= msg %></li>
    <% end %>
    </ul>
  </div>
<% end %>
```

视图片段引入了几个新的Rails和Ruby知识，包括Rails错误（erros）对象的两个方法。一个方法是count，返回错误的数量：

```ruby
>> user.errors.count
=> 2
```
另一个新方法是any?，它和empty?是互补的一对方法：
```ruby
>> user.errors.empty?
=> false
>> user.errors.any?
=> true
```
这里我们又看到empty?方法，我们在4.2.3节中介绍字符串的时候也看到过，它在Rails的errors对象上也工作，当对象为空时返回true，否则false。any?方法和empty?方法正好相反，假如有任何元素都会返回true，否则返回false。（顺便提一下，所有这些方法--count，empty?和any?--Ruby数组也有这些方法。我们将在11.2节里开始好好使用它们）

另一个新点子是pluralize文本辅助方法。默认情况下它在控制台里是不可用的，但是我们可以通过包含ActionView::Helpers::TextHelper模块来显示的调用。
```ruby
>> include ActionView::Helpers::TextHelper
>> pluralize(1, "error")
=> "1 error"
>> pluralize(5, "error")
=> "5 errors"
```
我们看到pluralize带一个整形参数，然后根据第一个参数返回第二个参数的正确的复数形式。该方法的底层通过强大的变形能力实现，它知道怎么把许多单词变成复数，其中包括许多不规则的单词的复数变换：

```ruby
>> pluralize(2, "woman")
=> “2 women”
>> pluralize(3, "erratum")
=> "3 eerata"
```
由于使用了pluralize，我们的代码
```
<%= pluralize(@user.errors.count, "error") %>
```
会依据错误数量返回“0 errors”，"1 error", "2 errors", 等等，避免了错误信息的语法不正确，例如"1 errors"（在应用和网页常见的错误）。

注意代码清单7.19包含id为error_explanation的CSS标记是为了方便格式化错误信息。（回忆5.1.2节，CSS id使用井号）。另外，在无效的提交后Rails自动把错误打包进
类名为field_with_errors的div里。这些标签允许我们使用代码清单7.20的SCSS格式化错误信息，我们使用了Sass的@extend函数来包含Bootstrap定义的has-error类。

```scss
代码清单7.0：为了格式化错误信息的CSS。
# app/assets/stylesheets/custom.css.scss
 .
.
.
/* forms */
.
.
.
#error_explanation {
  color: 红色;
  ul {
    color: 红色;
    margin: 0 0 30px 0;
  }
}

.field_with_errors {
  @extend .has-error;
  .form-control {
    color: $state-danger-text;
  }
}
```

有了代码清单7.18和7.19、7.20里的代码，当提交失败的注册信息时对用户有帮助的错误信息就会显示，如图7.18。因为这些错误信息是基于模型的有效性验证生成的，所以一旦你改变了有效性验证，例如，email地址格式或者密码的最小长度等，当用户提交的信息不满足这些要求时相应的错误信息就会被触发。

![图7.18：带错误信息的失败的注册](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/signup_error_messages_3rd_edition.png)

### 7.3.4 无效提交的测试
在那些还没有那种像Rails这样强大的自带完全测试的Web框架出现之前的日子里，开发者不得不手动测试表单。例如，为了手动测试注册页面，我们必须在浏览器里访问这个页面，然后交替提交有效或无效的数据，确认在各种情况下应用程序的行为是正确的。而且一旦应用程序改变我们不得不不断重复这个过程。这个过程是令人厌烦的而且很容易出错。

令人高兴的是，Rails允许我们为为表单写自动测试。本节我们将写一个测试用来确认当提交无效表单时我们的应用程序行为正确；在7.4.4节，我们将为有效提交写一个相应的测试。

为了开始写测试，我们首先需要为注册用户生成一个集成测试文件，我们叫users_signup(采用资源名称的复数是控制器的惯例）：

```terminal
$ rails generation integration_test users_signup
  invoke  test_unit
      create    # test/integration/users_signup_test.rb
```
(我们将使用和在7.4.4节同样的文件来测试有效的注册）

我们的测试的主要目的是确认当提交信息无效时点击注册按钮不会创建新用户。（为错误信息写测试留为练习（7.7节））实现它的方法是检查用户的数量，然后在后台我们的测试将使用count方法，在每个Active Record类都可用的方法，包含User：

```ruby
$ rails console
>> User.count
=> 0
```
这里User.count是0，因为我们在7.2节的开始重置了数据库。如在5.3.4节，我们将使用assert_select来测试相关页面的HTML元素，细心核对不会在将来改变的元素。

我们将通过使用get访问注册路径：
```ruby
get signup_path
```

为了测试表格提交，我们需要发出POST请求到users_path(表7.1），我们通过post函数实现：

```ruby
assert_no_difference 'User.count' do
  post users_path, user: { name:  "",
                           email: "user@invalid",
                           password:              "foo",
                           password_confirmation: "bar" }
end
```

这里我们为create动作传递了User.new需要的参数哈希params[:user]（代码清单7.24）。通过字符串参数‘User.count’打包在assert_no_differnce方法里的post，我们设计比较User.count在运行assert_no_difference的块之前和之后的变化。这和记录用户的数目，post数据，然后确认用户数不变是一样的：

```ruby
before_count = User.count
post users_path, ...
after_count  = User.count
assert_equal before_count, after_count
```

尽管它们的效果是相同的，但使用assert_no_difference更清晰、更地道。

值得一提的是，上面的get和post是严格来说没有关系的，实际上post到用户路径之前，get注册路径是不必要的。我偏好包含两步。不过，两个都是为了概念清晰和认真检查注册表单渲染没有出错。

把上面的想法放在一起就产生了代码清单7.21里的测试。我们也包含了对asser_template的调用，来检查失败提交重新渲染了new动作。添加代码来确认错误信息的测试留下来作为练习（7.7节）。

```ruby
代码清单7.1：无效注册的测试。绿色
# test/integration/users_signup_test.rb
require 'test_helper'

class UsersSignupTest < ActionDispatch::IntegrationTest
  test "invalid signup information" do
    get signup_path
    assert_no_difference 'User.count' do
      post users_path, user:{ name: "",
                    email:"user@invalid",
                    password:       "foo",
                    password_confirmation: "bar"}
    end
    assert_template 'users/new'
  end
end
```

因为我们在集成测试前写好了代码，测试集应该是绿色的：

```
代码清单7.2： 绿色
$ bundle exec rails test
```

## 7.4 成功的注册

处理完无效的表单提交，现在是时候通过真正地把一个新用户（假如有效）保存到数据库来完成我们的注册表单了。首先我们尝试保存用户；假如保存成功了，用户的信息会被自动写进数据库，那么我们就重定向浏览器显示用户信息页面（和友好的问候），如图7.19的原型。假如它失败了，我们简单地回到7.3节。

![图7.19：成功注册的原型](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/signup_success_mockup_bootstrap.png)

### 7.4.1 完成注册表单
为了让注册表单工作，我们需要在代码清单7.17里添加合适的代码取代那部分注释。现在，我们提交有效的表单会失败。如图7.20表明的一样，这是因为Rails动作的默认行为是渲染相应的视图，但是目前我们还没有与create动作相对应的视图模板。

![图7.20：有效注册提交返回的错误页面](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/valid_submission_error.png)

与渲染创建用户成功的页面相反，当用户创建成功后我们将重定向到不同的页面。尽管重定向到root_path也可以工作，我们还是按照惯例重定向到新创建的用户的个人信息页面。正如redirect_to方法说明的，我们用它来实现重定向。如代码清单7.23所示：

```ruby
代码清单7.3：带save和redirect的用户创建动作。
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  .
  .
  .
  def create
    @user = User.new(user_params)
    if @user.save
      redirect_to @user
    else
      render 'new'
    end
  end

  private

    def user_params
      params.require(:user).permit(:name, :email, :password,
                                   :password_confirmation)
    end
  end
```

注意我们用的是
```ruby
redirect_to @user
```
它和下面的代码是等价的
```ruby
redirect_to user_url(@user)
```
这是因为Rails会自动推断代码redirect_to @user实际想要重定向到user_url(@user)。

### 7.4.2 flash

在添加了代码清单7.23里的代码后，我们的注册表单其实已经可以工作了，但是在通过浏览器提交有效的注册信息之前，我们先为网站再添加一点点缀：在后续页面显示信息（在这个例子，欢迎新用户加入我们的网站），然后在访问其他页面或刷新页面后不再显示。

显示临时信息的Rails之道是使用一个特殊的方法flash，我们可以把它当做哈希。Rails应用程序中flash中键名为:success的用来保存成功时的消息（代码清单7.24）。

```ruby
代码清单7.4：为注册页面添加flash信息。
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  .
  .
  .
  def create
    @user = User.new(user_params)
    if @user.save
      flash[:success] = "Welcome to the Sample App!"
      redirect_to @user
    else
      render 'new'
    end
  end

  private

    def user_params
      params.require(:user).permit(:name, :email, :password,
                                   :password_confirmation)
    end
end
```
通过flash赋值，现在到了在重定向后在页面显示信息的时候了。我们的方法是遍历flash，然后把所有相关信息都插入网站布局文件。你可能想起4.3.3节中控制台的例子，我们看看怎样遍历flash：

```terminal
$ rails console
>> flash = { success: "It worked!", danger: "It failed." }
=> {:success=>"It worked!", danger: "It failed."}
>> flash.each do |key, value|
?>   puts "#{key}"
?>   puts "#{value}"
>> end
success
It worked!
danger
It failed.
```
通过学习这个方法，我们准备使用以下代码在全站显示flash内容：
```erb
<% flash.each do |message_type, message| %>
  <div class="alert alert-<%= message_type %>"><%= message %></div>
<% end %>
```
（HTML和ERb组成的代码是很丑的；把它变得漂亮点是本章7.7节的作业）这里内嵌Ruby
```erb
alert-<%= message_type %>
```
创建了和信息类型相应的CSS类，:success信息的类是
```css
alert-success
```
(:success键是符号，但是内嵌Ruyb自动在把它插入模板时会把它转换称字符串“success”）对每个键使用不同的类允许我们为不同消息应用不同的样式。例如，在8.1.4节我们将使用flash[:danger]来表明失败的登陆尝试。（实际上，我们已经使用过一次alert-danger，在代码清单7.19里用它来为错误信息添加样式）。Bootstrap CSS支持为四个这样的flash类（success，info，warning和danger）添加样式。我们在开发Sample App的过程中这几个样式都会用到。

因为这些信息也会被插入模板，
```ruby
flash[:success] = "Welcome to the Sample App!"
```
所以最后得到的完整的HTML结果为
```html
<div class="alert alert-success">Welcome to the Sample App!</div>
```
把上面讨论的内嵌Ruby代码放进站点布局文件，结果如代码清单7.25所示
```erb
代码清单7.5： 为网站布局添加flash变量的内容。
# app/views/layouts/application.html.erb
<!DOCTYPE html>
<html>
  .
  .
  .
  <body>
    <%= render 'layouts/header' %>
    <div class="container">
      <% flash.each do |message_type, message| %>
        <div class="alert alert-<%= message_type %>"><%= message %></div>
      <% end %>
      <%= yield %>
      <%= render 'layouts/footer' %>
      <%= debug(params) if Rails.env.development? %>
    </div>
    .
    .
    .
  </body>
</html>
```
### 7.4.3 第一次注册

我们可以通过在我们的网站上注册第一个用户，name是“Rails Tutorial”，email为“example@railstutorial.org”（图7.21）来看看我们工作了半天的效果。效果页面（图7.22）显示友好的注册成功的信息，使用了success类很好看的绿色，它来自于5.1.2节引入的Bootstrap CSS框架。（假如提示错误，说email地址已经被占用了，确保你已经运行过7.2节里提到的的db:migrate:reset Rails任务，然后重启开发环境的网页服务器。）然后，重新刷新用户显示页面，flash信息就像约好的一样消失了（图7.23）。

![图7.1：为第一次注册填充信息](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/first_signup.png)

![图7.22：成功的用户注册成功页面，带flash信息](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/signup_flash_3rd_edition.png)

![图7.23：浏览器重载后没有了flash信息的简介页面](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/signup_flash_reloaded_3rd_edition.png)

我们现在看看我们的数据库，再次确认新用户已经妥妥地保存到数据库了：

```terminal
$ rails console
>> User.find_by(email: "example@railstutorial.org")
=> #<User id: 1, name: "Rails Tutorial", email: "example@railstutorial.org",
created_at: "2014-08-29 9：3：17", updated_at: "2014-08-29 9：3：17",
password_digest: "$2a$10$zthScEx9x6EkuLa4NolGye6O0Zgrkp1B6LQ12pTHlNB...">
```

### 7.4.4 有效提交测试
在我们继续下一节之前，我们为有效提交写个测试，通过它来确认我们应用程序以及回归测试。和7.3.4节无效提交测试一样，我们的主要目的是确认数据库的内容。在这个例子，我们想要提交有效信息，然后确认用户被创建。与代码清单7.21类似，它使用了
```ruby
assert_no_difference 'User.count' do
  post users_path, ...
end
```
这里我们使用和它相对应的assert_difference方法：

```ruby
assert_difference ‘User.count', 1 do
  post_via_redirect users_path, ...
end
```

和assert_no_difference一样，第一个参数是字符串’User.count'，我们安排User.count在运行完assert_difference块内容前后进行比较。第二个（可选）参数说明了不同的程度（在这个例子，1）。

把assert_difference合并到代码清单7.21的文件中产生了如代码清单7.26里显示的测试。注意我们使用post_via_redirect变量来post到用户的路径。这个简单的设计模仿了提交后的重定向，即渲染'users/show'模板。（为flash也写一个测试可能是个不错的想法，让我们把它作为本章的练习（7.7节））
```ruby
代码清单7.6： 测试有效的注册。 绿色
# test/integration/users_signup_test.rb
require 'test_helper'

class UsersSignupTest < ActionDispatch::IntegrationTest
  .
  .
  .
  test "valid signup information" do
    get signup_path
    assert_difference 'User.count', 1 do
      post_via_redirect users_path, user: { name:  "Example User",
                                            email: "user@example.com",
                                            password:              "password",
                                            password_confirmation: "password" }
    end
    assert_template 'users/show'
  end
end
```

注意代码清单7.26也测试了在注册成功后渲染了用户信息显示页面。为了让它工作，Users路由（代码清单7.3）、Users的show动作（代码清单7.5）和show.html.erb视图（代码清单7.8）都可以正确地工作，因此，这行
```ruby
assert_template 'users/show'
```
是对几乎和用户信息页面相关的每件事情的敏感性测试。这类端到端对重要特性的覆盖阐明了为什么集成测试是如此有用。

## 7.5 专家级部署

现在我们已经有了一个可用的注册页面，是时候部署我们的应用程序了，该是让它在生产环境中工作了。尽管我们在第三章就开始部署我们的应用程序，但这是第一次它真的做了些事情，所以我们将抓住这次机会来次专家级的部署。具体来说，我们先添加一个重要的特性到生产应用程序来让注册更安全，然后我们用更适合真实世界的网站服务器来替代默认的服务器。

为了做好部署的准备，现在你应该把这些改动合并到master分支：

```terminal
$ git add -A
$ git commit -m "Finish user signup"
$ git checkout master
$ git merge sign-up
```

### 7.5.1 使用SSL
当提交这章开发的注册表格是，姓名、email地址和密码都是通过网络来传输的，因此很容易被拦截。这是我们的应用程序潜在的严重安全缺陷，要解决这个问题的方法是使用[安全套接字层](http://en.wikipedia.org/wiki/Transport_Layer_Security)
在它离开本地浏览器前加密所有相关信息。尽管我们只是在注册页面使用SSL，它也可以很容易地就扩大到整个站点级别，实现安全的用户登陆（第八章），也可以让我们的网站对在8.4节讨论的session劫持的严重弱点免疫。

要启用SSL只需要取消production.rb里中一行代码的注释，production.rb为生产应用程序的配置文件。如代码清单7.27显示，我们所需要做的是设置config变量来强迫在生产环境里使用SSL。
```ruby
代码清单7.7：配置应用程序在生产环境使用SSL。
# config/environments/production.rb
Rails.application.configure do
  .
  .
  .
  # Force all access to the app over SSL, use Strict-Transport-Security,
  # and use secure cookies.
  config.force_ssl = true
  .
  .
  .
end
```

在这里，我们需要在远程服务器上设置SSL。设置生产网站使用SSL需要付款和为你的域名配置SSL证书，有许多工作需要我们完成。不过，很幸运地是我们这里不需要：作为在Heroku上运行的应用程序（如Sample App），我们可以依附在Heroku的SSL证书上。因此当我们在7.5.2节部署应用程序时，SSL将自动启用。（假如你想要在自定义域名上运行SSL，例如www.example.com，参考[Heroku的关于SSL的页面](http://devcenter.heroku.com/articles/ssl)

### 7.5.2 在生产环境使用Unicorn

已经添加了SSL，我们现在需要配置我们的应用程序使用适合生产环境的应用程序的网页服务器。默认Herok使用WEBrick，纯ruby写的网页服务器，它很容易设置和运行，但是不擅长处理大流量。所以WEBrick不适合在生产环境使用，因此我们将使用Puma来替代WEBrick，Puma是一个有能力处理大量请求的HTTP服务器。

为了添加新的网页服务器，我们简单地学习[Heroku Puma文档](https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server)。因为Rails 5默认使用的服务器就是Puma，所以可以跳过其中把gem puma加入Gemfile的步骤。

```ruby
代码清单7.28：确认在Gemfile中添加了gem puma。
# Gemfile
source 'https://ruby.taobao.org'
.
.
.
group :production do
  gem 'pg',             '0.17.1'
  gem 'rails_12factor', '0.0.2'
end
```
因为我们配置了Bundler不安装生产环境的gem（3.1节），代码清单7.28不会添加任何开发环境中才需要包含的gem，但是我们仍然需要运行Bundler来更新Gemfile.lock:

```
$ bundle install
```
下一步是创建名为config/puma.rb的文件，然后加入代码清单7.29的内容。代码清单7.29里的代码直接来自于[Heroku文档](https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server)，不需要理解它。

```ruby
代码清单7.9： 为生产环境的服务器写的配置文件。
# config/puma.rb
workers Integer(ENV['WEB_CONCURRENCY'] || 2)
threads_count = Integer(ENV['MAX_THREADS'] || 5)
threads threads_count, threads_count

preload_app!

rackup      DefaultRackup
port        ENV['PORT']     || 3000
environment ENV['RACK_ENV'] || 'development'

on_worker_boot do
  # Worker specific setup for Rails 4.1+
  # See: https://devcenter.heroku.com/articles/
  # deploying-rails-applications-with-the-puma-web-server#on-worker-boot
  ActiveRecord::Base.establish_connection
end
```

最后，我们需要创建所谓的Procfile来告诉Heroku来在生产环境来运行Puma进程，如代码清单7.30所示。Procfile应该在你应用程序的根目录（例如，和Gemfile同一个目录）。

```ruby
代码清单7.30：为Puma定义Procfile。
# ./Procfile
web: bundle exec puma -C config/puma.rb
```

随着生产环境的网页服务器配置结束，我们准备提交和部署：

```terminal
$ bundle exec rails test
$ git add -A
$ git commit -m "Use SSL and the Puma webserver in production"
$ git push
$ git push heroku
$ heroku run rails db:migrate
```
注册表单现在可用了，成功注册的结果如图7.24所示。注意图7.24里https://的显示和在地址栏锁一样的标志，它表示SSL工作了。

![图7.24：在互联网上的注册页面](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/signup_in_production_3rd_edition.png)

### 7.5.3 Ruby版本数字
当部署到Heroku，你可能会得到一个像这样的警告：
```ruby
###### WARNING:
       You have not decla红色 a Ruby version in your Gemfile.
       To set your Ruby version add this line to your Gemfile:
       ruby '2.1.5'
```

经验显示，按照本书的水平，包含这样一个严格的Ruby版本数字的代价远超过受到的益处，所以你现在应该先忽视这个警告。主要问题是用最近的版本保持你的Sample App和系统同步可能非常不方便，而且使用确切的Ruby版本也不会有什么不同。然而，你应该记住，你应该在Heroku上运行重要使命的应用程序，因此在Gemfile指明确Ruby版本是被推荐得，它可以确保在开发和生产环境最大程度的兼容。

## 7.6 结语
能注册用户是我们应用程序的一个明显的里程碑。尽管Sample App依然没有完成任何有用的事情，但是我们已经为未来所有的开发奠定了基础。在第八章，我们将通过允许用户登陆和退出完善网站的授权机制。在第九章，我们允许用户更新他们的账户信息。我也将允许网站管理员删除用户，从而完成表7.1里的用户资源所有的REST动作。

### 7.6.1 在这章我们学习了什么
* Rails通过debug方法显示有用的调试信息
* Sass mixin允许捆绑一组CSS规则，在多个地方使用
* Rails自带三个开发环境：development, test和production。
* 我们能把用户当作资源使用标准的REST URL集来交互
* Gravatar提供了显示用户头像方便的方法
* form_for辅助方法常常用户创建和ApplicationRecord对象交互的表格
* 注册失败后渲染新用户页面，自动显示ApplicationRecord定义的错误信息
* 注册成功后在数据库里创建用户，然后重定向到用户信息页面，并且显示欢迎信息
* 我们能使用教程测试来确认表单提交行为和回归测试
* 我们能配置我们的生产应用程序使用SSL来安全地通讯，使用高性能的Puma网页服务器

## 7.7 练习
1. 确认代码清单7.31里的代码，允许在7.1.4节里定义的gravatar_for辅助方法带一个可选的尺寸参数，允许在视图里使用gravatar_for user, size: 50 这样的代码。（我们将在9.3.1节使用提高过的辅助方法）

2. 为在7.18里的失败信息写一个测试。细节可以自己决定；建议使用代码清单7.32的模板。
3. 为7.4.2节的flash实现写个测试。细节可以自己决定；建议超级小的模板如7.33，你应该通过替换FILL_IN完成。（即使测试正确的键，更少的文本也可能导致测试容易打碎，所以我偏好仅测试flash非空）
4. 如在7.4.2节里提到的，在代码清单7.25里flash的HTML是丑陋的，通过运行测试集来确认代码清单7.34更整洁的代码，它使用了Rails content_tag辅助方法，也可以工作。

```ruby
代码清单7.1： 为gravatar_for方法添加可选哈希参数。
# app/helpers/users_helper.rb
 module UsersHelper

  # Returns the Gravatar for the given user.
  def gravatar_for(user, options = { size: 80 })
    gravatar_id = Digest::MD5：:hexdigest(user.email.downcase)
    size = options[:size]
    gravatar_url = "https://secure.gravatar.com/avatar/#{gravatar_id}?s=#{size}"
    image_tag(gravatar_url, alt: user.name, class: "gravatar")
  end
end
```
```ruby
代码清单7.2：测试错误信息的模板。
# test/integration/users_signup_test.rb
 require 'test_helper'

class UsersSignupTest < ActionDispatch::IntegrationTest

  test "invalid signup information" do
    get signup_path
    assert_no_difference 'User.count' do
      post users_path, user: { name:  "",
                               email: "user@invalid",
                               password:              "foo",
                               password_confirmation: "bar" }
    end
    assert_template 'users/new'
    assert_select 'div#<CSS id for error explanation>'
    assert_select 'div.<CSS class for field with error>'
  end
  .
  .
  .
end
```
```ruby
代码清单7.3：测试flash的模板。
# test/integration/users_signup_test.rb
 require 'test_helper'
  .
  .
  .
  test "valid signup information" do
    get signup_path
    assert_difference 'User.count', 1 do
      post_via_redirect users_path, user: { name:  "Example User",
                                            email: "user@example.com",
                                            password:              "password",
                                            password_confirmation: "password" }
    end
    assert_template 'users/show'
    assert_not flash.FILL_IN
  end
end
```
```ruby
代码清单7.4：在网站布局文件中使用content_tag。
# app/views/layouts/application.html.erb
 <!DOCTYPE html>
<html>
      .
      .
      .
      <% flash.each do |message_type, message| %>
        <%= content_tag(:div, message, class: "alert alert-#{message_type}") %>
      <% end %>
      .
      .
      .
</html>
```

