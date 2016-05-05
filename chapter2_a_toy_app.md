#<span id="2">2 玩具网站</span>

这章我们将用一个名字叫做玩具（toy）的应用程序来炫耀一下Rails的强大。通过使用**scaffold**快速生成应用程序， 目的是让我们对Ruby on Rails编程（和网页开发的基本原理）有个总体的认知。它会自动创建大量函数。如同我们在[注1.2(#b1.2)]中讨论过的一样，从第三章开始我们将使用跟第二章完全相反的方式来开发网站，也就是我们不再使用脚手架来生成代码，而是从零开始，循序渐进地开发网站，每一个新概念都会有详细的讲解，但是为了能够快速浏览（还有创作的成就感），脚手架在这些方面还是有着无可替代的优势。玩具网站允许我们通过URL与它进行交互，还让我们直观地了解到Rails应用的内在结构，也包括Rails偏好的REST架构的首次演示。

和本书后续的网站相同，玩具网站将包含用户和用户发布的微博（类似于迷你版的新浪微博）。功能模块还需要在后续开发，而且很多步骤看起来像变魔术，不过不用担心：后面的完整的示例程序（sample网站）将会从零开始开发和这个网站功能差不多的新的网站，从[第三章](#3)开始，而且我还会提供大量的资料供后续参考。而现在，你所需要的是耐心和一点信心——这本书的重点在于带你透过现象看本质，通过脚手架这个例子对Rails有更深入的理解。

##<span id="2.1">2.1 前期准备</span>

在这一节，我们对玩具网站做一个概要设计。如同在[1.3](#1.3)中一样，我们将从创建这个应用的骨架入手，还是用`rails new`命令，同时要记得加上Rails的版本号：

```
$ cd ~/workspace
$ rails _5.0.0_ new toy_app
$ cd toy_app/

```

如果运行了以上的命令出现类似这样的错误“Could not find 'railties'”（找不到rails相关资源），那说明你没有安装好正确的Rails版本，这时你要好好确认一下是否严格按照[命令清单1.1](#l1.1)的步骤来安装Rails。（如果你使用的是我们推荐的云IDE（[1.2.1](#1.2.1)）注意这第二个应用项目是可以和上一个应用建立在同一个工作空间里的，不需要再重新创建一个新的工作空间。有时候你新建了一个新的项目但是看不到相关文档，这时候你需要点击右上角的齿轮状图标，选择“Refresh File Tree”（刷新文件目录树）。）

下一步，我们用文本编辑器将Gemfile文档更新如[代码清单2.1](#l2.1)，以便Bundler命令使用。

<code><span id="l2.1" style="font-size:12px; font-weight:bold; font-family:''">代码清单2.1：toy app使用的Gemfile文档。</span></code> 

```
source 'https://rubygems.org'

gem 'rails',        '5.0.0'
gem 'sass-rails',   '5.0.2'
gem 'uglifier',     '2.5.3'
gem 'coffee-rails', '4.1.0'
gem 'jquery-rails', '4.0.3'
gem 'turbolinks',   '2.3.0'
gem 'jbuilder',     '2.2.3'
gem 'sdoc',         '0.4.0', group: :doc

group :development, :test do
  gem 'sqlite3',     '1.3.9'
  gem 'byebug',      '3.4.0'
  gem 'web-console', '2.0.0.beta3'
  gem 'spring',      '1.1.3'
end

group :production do
  gem 'pg',             '0.17.1'
  gem 'rails_12factor', '0.0.2'
end
```
注意，[代码清单2.1](#l2.1)其实和[代码清单1.14](#l1.14)是一样的。

如同[1.5](#1.5)一样，接下来我们将使用`--without production`选项，只安装本地开发所需的gem，而不安装生产环境所需的gem：

```
$ bundle install --without production

```

最后，我们将玩具网站纳入Git版本管理中：

```
$ git init
$ git add -A
$ git commit -m "Initialize repository"

```

点击Bitbucket（[图2.1](#p2.1)）上的“Create”按钮，新建一个[新仓库](https://bitbucket.org/repo/create)，然后将代码推送到这个远端仓库中。

```
$ git remote add origin git@bitbucket.org:<username>/toy_app.git
$ git push -u origin --all # 第一次推送代码到远程仓库

```

<span id="p2.1">![create_demo_repo_bitbucket](http://i.imgur.com/3swZBtG.png)</span>  
图2.1 在Bitbucket上新建一个toy app所需的托管仓库

最后，来部署验证一下，我建议完全按照之前的例程“hello,world!”中的步骤操作，请回去参考[代码清单1.8](#l1.8)和[代码清单1.9](#l1.9)。然后将改动提交，并推送到Heroku上：

```
$ git commit -am "Add hello"
$ heroku create
$ git push heroku master

```

（如同在[1.5](#1.5)一样，你可能会看到很多警告信息，现在你完全可以无视他们。我们会在[7.5](#7.5)）处理这些信息。）除了Heroku 应用的地址不一样之外，其他的应该都和[图1.18](#p1.18)一样。

现在我们已经准备好开始开发这个玩具网站了。按照正常流程，开发Web应用的第一步应该是创建数据模型（data model），它是应用程序所需结构的体现。在我们这个项目中，玩具网站将会是一个微博，只有用户和微博。所以，我们将从用户模型开始开发我们的app（[2.1.1](#2.1.1)），然后再添加一个微博模型（[2.1.2](#2.1.2)）。


###<span id="2.1.1">2.1.1 玩具网站的用户模型</span>

在网上有多少种类型的注册表单，就对应有多少种类型的用户数据模型；我们将从最简单的开始。我们的网据网站用户将会有一个身份验证号（id），用整数类型（integer）来表示，一个公共可见的用户名（name），用字符串类型（string）来表示，还有一个email地址（也是字符串string类型），email地址也可作为用户名使用。用户数据模型可以总结为[图2.2](#p2.2)所示：

<span id="p2.2">![demo_user_model](http://i.imgur.com/gihl57a.png)</span>  
图2.2 用户数据模型

我们在[6.1.1](#6.1.1)中将会发现，[图2.2](#p2.2)中的“users”标签对应数据库中的表单名，而“id”，“name”，“email”将对应表中的列名。

###<span id="2.1.2">2.1.2 玩具网站的微博数据模型</span>

微博数据模型的核心甚至比用户数据模型还要简单：微博数据模型只需一个id号和存放微博内容的content字段（类型为“text”）。但是我们还需要一个额外的复杂一些的字段：为了把每一篇微博和它的拥有者联系起来，我们要用到“user_id”字段，如[图2.3](#p2.3)所示。

<span id="p2.3">![demo_micropost_model](http://i.imgur.com/0zVNzN3.png)</span>  
图2.3 微博数据模型

我们将在[2.3.3](#2.3.3)中了解到“user_id”字段的作用（[11](#11)中的介绍更详细），这个字段可以让我们很容易实现一个用户拥有多个关联微博的想法。

##<span id="2.2">2.2 用户资源</span>

在这一节里，我们将实现[2.1.1](#2.1.1)描述的用户数据模型，同时还要为这个模型实现一个web接口界面。这两者的结合就组成了我们的用户资源（Users resource），我们可以把用户（users）想象成一个对象（object），这个对象可以被创建，读取，更新，以及删除等操作，这些操作都可以通过[HTTP协议](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)在web页面上实现。正如在前面的简介中提到的，我们的用户资源将由Rails脚手架程序创建。我劝你现在还是不要太深究这些由脚手架生成的代码，因为在这个阶段，它只会让你感到更迷惑。

通过传递**scaffold**到**rails generate**脚本程序里生成Rails脚手架。**scaffold**命令的参数是资源名的单数形式（在这里，是“User”），同时还可以加上一些数据模型的属性的作为参数：

```
$ rails generate scaffold User name:string email:string
      invoke  active_record
      create    db/migrate/20140821011110_create_users.rb
      create    app/models/user.rb
      invoke    test_unit
      create      test/models/user_test.rb
      create      test/fixtures/users.yml
      invoke  resource_route
       route    resources :users
      invoke  scaffold_controller
      create    app/controllers/users_controller.rb
      invoke    erb
      create      app/views/users
      create      app/views/users/index.html.erb
      create      app/views/users/edit.html.erb
      create      app/views/users/show.html.erb
      create      app/views/users/new.html.erb
      create      app/views/users/_form.html.erb
      invoke    test_unit
      create      test/controllers/users_controller_test.rb
      invoke    helper
      create      app/helpers/users_helper.rb
      invoke      test_unit
      create        test/helpers/users_helper_test.rb
      invoke    jbuilder
      create      app/views/users/index.json.jbuilder
      create      app/views/users/show.json.jbuilder
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/users.js.coffee
      invoke    scss
      create      app/assets/stylesheets/users.css.scss
      invoke  scss
      create    app/assets/stylesheets/scaffolds.css.scss
```

命令里包含了`name:string`和`email:string`，这样我们就可以实现[图2.2](#p2.2)中的用户模型结构了。（注意我们不需要将`id`字段包含在命令中，因为Rails会我们自动生成**id**，并把**id**作为数据表的主键（primary key））

为了运行玩具网站，首先我们要使用Rake来迁移（migrate）数据库（详见[注2.1](#b2.1)）：

```
$ bundle exec rake db:migrate
==  CreateUsers: migrating ====================================================
-- create_table(:users)
   -> 0.0017s
==  CreateUsers: migrated (0.0018s) ===========================================
```

这是使用我们新的users数据模型来更新数据库。（我们会在[6.1.1](#6.1.1)学习更多关于数据库迁移（database migration）的知识。）注意，为了确保所使用的Rake命令版本与我们的Gemfile文件中的一致，我们需要使用`bundle exec`来运行``rake`。在很多系统里，包括我们的云IDE，都可以省略`bundle exec`，但是在有些系统里这个命令是必须的，所以为了完整性，我会一直使用`bundle exec`。

接下来重新打开一个终端（[图1.7](#p1.7)）使用以下的命令启动Web服务：  
```
$ rails server -b $IP -p $PORT    # 如果运行在本机上只需要输入`rails sever`

```

现在我们的玩具网站应该可以在本地运行起来了，如同[1.3.2](#1.3.2)描述的情况一样。（如果你使用云IDE，请记住要在一个新的浏览器窗口里运行我们的服务器，而不是在IDE里运行。）

#### 注2.1 Rake 命令

在传统的Unix系统里，“[make](http://en.wikipedia.org/wiki/Make_(software))”命令在将源代码编译成可执行程序的过程中扮演着非常重要的角色，很多计算机黑客甚至已经做到肌肉记忆了。

```
$ ./configure && make && sudo make install
```

在Unix系统里，这行命令一般用来编译代码（适用于Linux系统和Mac OS X系统）。

“Rake”就相当于Ruby语言的“make”，用Ruby写的类make语言。Rails使用Rake的范围很广，尤其在开发基于数据库的web应用要做的那些无数的小型管理任务时。`rake db:migrate`命令应该是最普遍的一条命令，但是还有其他类似的普遍命令；你可以看到一长串的数据库任务使用`-T db`：

```
$ bundle exec rake -T db
```

要查看所有可用的Rake任务，运行

```
$ bundle exec rake -T
```

这个列表看起来可能非常庞大，但是别担心，你不需要知道所有的内容（甚至也不需要知道太多）。在结束本书的阅读后，你将会了解到其中最重要的那些。

为了不让Rails初学者感到困惑，Rails 5已经把rake命令和rails命令整合为rails命令。因此，以上命令中的rake都可以用rails来替代。后面我们将统一使用rails命令，如果你要使用Rails 5之前的版本，请查阅相应的文档。

###<span id="2.2.1">2.2.1 用户演示</span>

如果我们在“/”（读做“斜杠”，[1.3.4](#1.3.4)有说明）访问根URL（http://localhost:3000/），我们还是会得到如[图1.9](#p1.9)所示的默认Rails页面，但是使用用户资源脚手架，我们同时也创建了很多用来操作用户的页面。例如，用来列出所有用户的页面是“[/users](http://localhost:3000/users)”，以及添加新用户的页面是“[/users/new](http://localhost:3000/users/new)”。这节剩余的部分让我们通过这些用户相关的页面来场旋风之旅。我们继续，[表格2.1](#t2.1)中的内容对我们接下来的学习应该有帮助，他显示了页面和URL之间的关系。

| URL        | 动作   |  作用  |
| :--------   | :-----  | :----  |
| /users     | index | 用来显示所有用户的页面|
| /users/1  | show  |显示用户id为1的用户页面|
| /users/new    | new |添加新用户的页面|  
| /users/1/edit | edit|用来修改用户id为1的用户的页面|

表格2.1 用户资源的页面和URL之间的关联

在这个应用中，我们将从显示所有用户信息的页面开始入手，也称为[index](http://localhost:3000/users)页面；正如你所意料的，初始化的页面里根本没有用户（[图2.4](#p2.4)）。

<span id="p2.4">![demo_blank_user_index_3rd_edition](http://i.imgur.com/Ukit2hH.png)</span>

图2.4 用户资源的初始index页面（[/users](http://localhost:3000/users)）

为了添加一个新用户，我们要访问[new](http://localhost:3000/users/new)页面，如[图2.5](#p2.5)所示。（因为我们在本地开发时，其地址要么是http://localhost:3000，或者是云端IDE分配的地址，接下来我就不再重复啰嗦这些地址了。）在[7](#7)，这个页面将会成为我们的注册页面。

<span id="p2.5">![demo_new_user_3rd_edition](http://i.imgur.com/rznX6DR.png)</span>

图2.5 用户的new页面（[/users/new](http://localhost:3000/users/new)）

我们在指定的方框内填写新用户的用户名，email地址后，再按一下“Create User”（新建用户）按钮，就可以新建一个新用户了。得到的结果就是用户详细信息（[show](http://localhost:3000/users/1)）页面，如[图2.6](#p2.6)所示。（绿色的欢迎信息是使用flash显示的（译者注：这里的flash和我们平常说的flash不是一回事，这里的flash只是Rails里面一个特殊的变量），我们将在[7.4.2](#7.4.2)学习它。）注意这里的URL是“[/users/1](http://localhost:3000/users/1)”；可能你已经注意到了，数字“1”就是来自[图2.2](#p2.2)中的用户id字段。在[7.1](#7.1)，这个页面将变成用户资料页面。

<span id="p2.6">![demo_show_user_3rd_edition](http://i.imgur.com/lxj9ZpY.png)</span>

图2.6 显示用户信息的页面（[/users/1](http://localhost:3000/users/1)）

要改变用户信息，我们需要访问“[edit](http://localhost:3000/users/1/edit)”（编辑）页面，如[图2.7](#p2.7)所示。修改了所需的内容后，我们单击“Update User”（更新用户）按钮，这样就可以更新玩具网站的用户信息了（[图2.8](#p2.8)）。（从[6](#6)开始，我们将看到这部分内容的更多细节，这个用户数据将会存储在后端的数据库中。）在[9.1](#9.1)我们将会为sample应用了添加用户信息的“edit/update”（编辑/更新）功能。

<span id="p2.7">![demo_edit_user_3rd_edition](http://i.imgur.com/QXMYkH4.png)</span>  
图2.7 用户信息编辑页面（[/users/1/edit](http://localhost:3000/users/1/edit)）


<span id="p2.8">![demo_update_user_3rd_edition](http://i.imgur.com/AxExfLh.png)</span>  
图2.8 用户信息更新后的页面

接下来我们重新访问“[new](http://localhost:3000/users/new)”页面来添加第二个用户，同时提交第二个用户的用户信息。得到的用户[index](http://localhost:3000/users)页面将如[图2.9](#p2.9)所示。在[7.1](#7.1)我们将会美化这个用户主页，用来显示所有的用户信息。

<span id="p2.9">![demo_user_index_two_3rd_edition](http://i.imgur.com/9oB73yb.png)</span>  
图2.9 用户index页面显示第二个用户信息（[/users](http://localhost:3000/users)）

在经过了创建，显示，编辑这些步骤以后，接下来我们要删除用户（[图2.10](#p2.10)）。你需要用[图2.10](#p2.10)里的“destroy”按钮来删除第二个用户，刷新页面，只剩下一个用户了。（如果你删不掉，你得查看一下你的浏览器是否支持JavaScript；Rails使用JaveScript来发出删除用户的请求。）[9.4](#9.4)将会在我们的sample应用中添加用户删除功能，这个功能只有特殊的管理组用户才能使用。


<span id="p2.10">![demo_destroy_user_3rd_edition](http://i.imgur.com/SOwTXWy.png)</span>  
图2.10 删除一个用户

###<span id="2.2.2">2.2.2 使用MVC</span>

既然我们已经快速浏览了用户资源的内容，接下来让我们用[1.3.3](#1.3.3)中提到的MVC架构（模型-视图-控制器）来验证一下用户资源里的一些特定部分。我们的想法是，通过一个典型的浏览器访问过程——访问用户主页（[/users](http://localhost:3000/users)），来了解一下MVC（[图2.11](#p2.11)）。

<span id="p2.11">![mvc_detailed](http://i.imgur.com/1oeWxYU.png)</span>  
图2.11 Rails中的MVC详细图解

以下是[图2.11](#p2.11)的步骤总结：  
1. 浏览器向“/users”发送一个URL请求；
2. Rails将“/users”路由到“Users controller”里的“index”动作；
3. “index”动作请求“User model”检索所有的用户（`User.all`）；
4. “User model”从数据库中拉取所有的用户信息；
5. “User model”将用户列表返回给控制器；
6. 控制器将用户赋值给`@users`变量，然后将这个变量传到index视图；
7. 视图使用内嵌的Ruby程序将页面渲染成HTML格式；
8. 控制器将HTML文件传回浏览器。

现在让我们深入研究一下以上的步骤。我们从浏览器发出请求——例如，在浏览器地址栏里输入URL或直接点击一个链接（图2.11的步骤1）。这个请求传送到Rails的路由（步骤2），路由基于URL决定将这个请求发送给某个合适的控制器动作来处理（[注3.2](#b3.2)列出了请求的类型）。[代码清单2.2](#l2.2)用来为用户资源生成从URL到控制器动作的映射；这份代码有效地建立起了[表格2.1](#t2.1)中URL和动作的对应关系。（这个奇怪的表示法**:users**是symbol，我们将在[4.3.3](#4.3.3)中进一步了解）。

<code><span id="l2.2" style="font-size:12px; font-weight:bold; font-family:''">代码清单2.2：Rails路由，其中定义了用户资源的路由规则。</span></code>   
`config/routes.rb`  
```
Rails.application.routes.draw do
  resources :users
  .
  .
  .
end

```

既然我们已经在查看路由文件了，那让我们把根路由指向用户主页，这样当访问“/”时就会显示“/users”页面。回忆一下，在[代码清单1.10](#l1.10)中，我们将下面的代码

```
# root 'welcome#index'

```

改成

```
root 'application#hello'

```

所以根路由指向了app控制器的hello动作。在现在的这个例子里，我们想要使用用户控制器（Users controller）的index动作（action），我们可以使用[代码清单2.3](#l2.3)来实现。（此时此刻，如果你再这一节开始的时候添加了hello动作，我建议将其从app控制器中删除。）

<code><span id="l2.3" style="font-size:12px; font-weight:bold; font-family:''">代码清单2.3：为users添加根路由。</span></code>   
`config/routes.rb`  
```ruby
Rails.application.routes.draw do
  resources :users
  root 'users#index'
  .
  .
  .
end

```

[2.2.1](#2.2.1)中浏览的页面，对应于用户控制器里的不同动作，这是相关动作的集合。使用脚手架生成的控制器显示在[代码清单2.4](#l2.4)中。注意`class UsersController < ApplicationController`这种写法，这是Ruby类继承的表示方法。（在[2.3.4](#2.3.4)我们会简要说明，在[4.4](#4.4)会详细说明）

<code><span id="l2.4" style="font-size:12px; font-weight:bold; font-family:''">代码清单2.4：用户控制器代码结构示意。</span></code>   
`app/controllers/users_controller.rb`  

```  
class UsersController < ApplicationController
  .
  .
  .
  def index
    .
    .
    .
  end

  def show
    .
    .
    .
  end

  def new
    .
    .
    .
  end

  def edit
    .
    .
    .
  end

  def create
    .
    .
    .
  end

  def update
    .
    .
    .
  end

  def destroy
    .
    .
    .
  end
end
```

你可能注意到了动作（action）的数量要多于页面的数量；`index`，`show`，`new`和`edit`等动作都对应于[2.2.1](#2.2.1)提到的页面，但是同时还有`create`，`update`，`destroy`等其他动作。这些动作一般不直接用来渲染页面（虽然ta们有这个能力）；实际上，他们主要的作用在于修改数据库中的用户信息。在[表格2.2](#t2.2)中列出了控制器的所有动作，Rails就是用这些动作来实现REST架构（[注2.2](#b2.2)），REST架构的意思是“表现层状态转移”（representational state transfer），他是由计算机科学家[ Roy Fielding](https://en.wikipedia.org/wiki/Roy_Fielding)提出来的。注意在[表格2.2](#t2.2)中在一些URL中有重叠；例如，用户`show`和`update`动作都对应到“/users/1”URL地址。他们之间的区别在于他们使用的[HTTP请求方法](http://en.wikipedia.org/wiki/HTTP_request#Request_methods)不同。我们将在[3.3](#3.3)学习关于HTTP请求方法的内容。


| HTTP请求        | URL地址   |  动作  |作用|
| :--------   | :-----  | :----  | :----|
| GET  | /users | index| 显示所有用户的页面|
| GET  | /users/1  |show| 显示id为1的用户的页面|
| GET  | /users/new |new| 创建新用户的页面  |
| POST | /users |create|创建一个新用户 |
| GET  | /users/1/edit|edit| 修改id为1的用户的页面|
| PATCH  | /users/1 | update| 更新id为1的用户|
| DELETE | /users/1 | destroy| 删除id为1的用户|  

表格2.2 [代码清单2.2](#l2.2)生成的RESTful架构路由表

    #### 注2.2 表现层状态转移（REST）

    如果你阅读过一些关于Ruby on Rails的web开发资料，你会看到很多提放都提到“REST”架构，这是“表现层状态转移”（REpresentational State Transfer）的缩写。REST是一个用来做分布式的网络系统和软件应用的架构，例如我们熟悉的万维网（www）和web应用开发。虽然REST的概念听起来比较抽象，但是在Rails应用环境中，REST意味着大部分的应用组件（例如用户和微博）都会被模型化成为资源，这些资源可被生成（create），可读取（read），可更新（update），并且也可删除（delete）——这些操作对应于[关系型数据库的CRUD操作](http://en.wikipedia.org/wiki/Create,_read,_update_and_delete)和[HTTP请求方法](https://en.wikipedia.org/wiki/HTTP_request#Request_methods)的四个基本操作：POST,GET,PATCH,DELETE。（我们将在[3.3](#3.3)学习HTTP请求的知识，尤其是在[注3.2](#b3.2)中更详细。）

     作为一个Rails的开发者，RESTful风格的开发方法将会帮你选择应该写哪个控制器和动作：你只需要使用可以建立，读取，更新，删除的资源来构建应用就可以了。在这个例子里，“用户”和“微博”就很直观，因为他们都很自然地会被当成资源来对待。在[12](#12)中，我们将会看到如何使用REST基本准则对一个微秒的问题进行建模，“跟随用户”，使用一种自然而便利的方法来实现。


为了验证用户控制器和用户模型之间的关系，让我们将注意力集中在一个简化后的index动作版本上，如同[代码清单2.5](#l2.5)所示。（脚手架代码又难看又不容易理解，所以我省略了一些不需要的细节。）

<code><span id="l2.5" style="font-size:12px; font-weight:bold; font-family:''">代码清单2.5：toy app简化版的用户index动作。</span></code>   
`app/controllers/users_controller.rb`  

```
class UsersController < ApplicationController
  .
  .
  .
  def index
    @users = User.all
  end
  .
  .
  .
end

```

这个index动作里有一行代码` @users = User.all`（[图2.11](https://www.railstutorial.org/book/toy_app#fig-mvc_detailed)的步骤3），这行代码的作用是请求用户模型从数据库中返回所有用户信息列表（[图2.11](https://www.railstutorial.org/book/toy_app#fig-mvc_detailed)的步骤4），然后将这个列表赋值给变量`@users`（[图2.11](https://www.railstutorial.org/book/toy_app#fig-mvc_detailed)的步骤5）。用户模型的代码在[代码清单2.6](#l2.6)里；虽然代码看起来很少，但是其实功能很强大，应该这里使用了类的继承机制（[2.3.4](#2.3.4)和[4.4](#4.4)）。具体来说，我们通过调用Rails的库“Active Record”，User.all就可以从数据库中返回所有的用户信息了。

<code><span id="l2.6" style="font-size:12px; font-weight:bold; font-family:''">代码清单2.6：toy app的用户模型。</span></code>   
`app/models/user.rb`  

```
class User < ActiveRecord::Base
end

```

一旦`@users`变量定义之后，控制器将会调用视图（view）（步骤6），如[代码清单2.7](#l2.7)所示。以`@`符号为开头的变量，我们称为实例变量（instance variables），在视图中的自动可用的变量；在这个例子中，[代码清单2.7](#l2.7)中的`index.html.erb`视图将会遍历`@users`列表，然后为每个用户生成一条HTML代码。
（请记住，现在你不需要完全明白这些代码的意思。这里是演示一下。）

<code><span id="l2.7" style="font-size:12px; font-weight:bold; font-family:''">代码清单2.7：用户index页面视图HTML代码。</span></code>   
`app/models/user.rb` 

```
<h1>代码清单 users</h1>

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Email</th>
      <th colspan="3"></th>
    </tr>
  </thead>

<% @users.each do |user| %>
  <tr>
    <td><%= user.name %></td>
    <td><%= user.email %></td>
    <td><%= link_to 'Show', user %></td>
    <td><%= link_to 'Edit', edit_user_path(user) %></td>
    <td><%= link_to 'Destroy', user, method: :delete,
                                     data: { confirm: 'Are you sure?' } %></td>
  </tr>
<% end %>
</table>

<br>

<%= link_to 'New User', new_user_path %>

```

视图将代码转换为HTML（步骤7），然后控制器将其回传给浏览器显示（步骤8）。


###<span id="2.2.3">2.2.3 用户资源的弊端</span>

虽然使用脚手架可以很直观的看到Rails的结构，但是由脚手架生成的用户资源也有一些弊端：

 * **没有数据验证功能。**我们的用户模型会无差别的接受任何数据，可能有些空的名字或者无效的email地址等。
 * **没有认证机制。**我们没有登录或注销操作，任何用户都可以进行操作。
 * **没有测试。**技术上来讲这个可能不对——脚手架里包含了一些基本测试——但是这些测试没有包含数据有效性，认证等测试，更不用说其他客户要求的测试了。
 * **没有页面样式和布局。**没有网站样式和导航。
 * **没有真正理解代码。**如果你能读懂脚手架的代码，那你可以不用阅读本书了。


##<span id="2.3">2.3 微博资源</span>

我们已经生成和浏览了用户资源，接下来要把微博资源关联进来。在这一节的学习过程中，我建议你将本节的微博资源相关元素与之前[2.2](#2.2)里的用户资源相关元素对比一下；你将会发现这两种资源在很多内容上很相似。通过这种重复的形式，我们可以更好地理解Rails应用的REST的架构——确实，在现在比较早的阶段就观察用户资源和微博资源的异同也是本章的主要目的之一。

###<span id="2.3.1">2.3.1 微博资源的微旅行</span>

如同用户资源一般，我们也要为微博资源生成脚手架代码，使用的命令是`rails generate scaffold`，在这里，我们使用[图2.3](#p2.3)所示来生成数据模型：

```
$ rails generate scaffold Micropost content:text user_id:integer
      invoke  active_record
      create    db/migrate/20140821012832_create_microposts.rb
      create    app/models/micropost.rb
      invoke    test_unit
      create      test/models/micropost_test.rb
      create      test/fixtures/microposts.yml
      invoke  resource_route
       route    resources :microposts
      invoke  scaffold_controller
      create    app/controllers/microposts_controller.rb
      invoke    erb
      create      app/views/microposts
      create      app/views/microposts/index.html.erb
      create      app/views/microposts/edit.html.erb
      create      app/views/microposts/show.html.erb
      create      app/views/microposts/new.html.erb
      create      app/views/microposts/_form.html.erb
      invoke    test_unit
      create      test/controllers/microposts_controller_test.rb
      invoke    helper
      create      app/helpers/microposts_helper.rb
      invoke      test_unit
      create        test/helpers/microposts_helper_test.rb
      invoke    jbuilder
      create      app/views/microposts/index.json.jbuilder
      create      app/views/microposts/show.json.jbuilder
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/microposts.js.coffee
      invoke    scss
      create      app/assets/stylesheets/microposts.css.scss
      invoke  scss
   identical    app/assets/stylesheets/scaffolds.css.scss
   
```

（如果看到Spring相关的错误，只需重新运行代码即可。）为了更新数据库以便使用新数据模型，我们需要运行如[2.2](#2.2)中的迁移命令：

```
$ bundle exec rails db:migrate
==  CreateMicroposts: migrating ===============================================
-- create_table(:microposts)
   -> 0.0023s
==  CreateMicroposts: migrated (0.0026s) ======================================

```

现在我们所处的创建微博资源阶段和在[2.2.1](#2.2.1)中建立用户资源阶段是一样的。你可能已经猜到了，脚手架生成器已经将微博资源的Rails路由规则做了更新，如同[代码清单2.8](#l2.8)中所示。如同用户资源一样，`resources :microposts`路由规则将微博URL地址映射到微博控制器对应的动作上，如[表格2.3](#t2.3)所示。

<code><span id="l2.8" style="font-size:12px; font-weight:bold; font-family:''">代码清单2.8：Rails路由，有一条微博资源的新规则。</span></code>   
`config/routes.rb` 

```ruby
Rails.application.routes.draw do
  resources :microposts
  resources :users
  .
  .
  .
end
```

| HTTP请求        | URL地址   |  动作  |作用|
| :--------   | :-----  | :----  | :----|
| GET  | /microposts | index| 显示所有微博的页面|
| GET  | /microposts/1  |show| 显示id为1的微博页面|
| GET  | /microposts/new |new| 创建新微博页面  |
| POST | /microposts |create|创建一篇新微博 |
| GET  | /microposts/1/edit|edit| 修改id为1的微博页面|
| PATCH  | /microposts/1 | update| 更新id为1的微博|
| DELETE | /microposts/1 | destroy| 删除id为1的微博|  
表格2.3 微博资源[代码清单2.8](#l2.8)生成的RESTful架构路由表

微博控制器（MicropostsController）代码摘要如[代码清单2.9](#l2.9)所示。请注意，除了名字不一样之外，[代码清单2.9](#l2.9)和[代码清单2.4](#l2.4)其实没什么区别。这是REST架构反映在两种资源之间的共同之处。

<code><span id="l2.9" style="font-size:12px; font-weight:bold; font-family:''">代码清单2.9：微博控制器代码摘要。</span></code>   
`app/controllers/microposts_controller.rb` 

```
class MicropostsController < ApplicationController
  .
  .
  .
  def index
    .
    .
    .
  end

  def show
    .
    .
    .
  end

  def new
    .
    .
    .
  end

  def edit
    .
    .
    .
  end

  def create
    .
    .
    .
  end

  def update
    .
    .
    .
  end

  def destroy
    .
    .
    .
  end
end

```

现在我们来新建几篇真正的微博，在新的微博页面上输入微博内容，如[图2.12](#p2.12)显示的[/microposts/new](http://localhost:3000/microposts/new)页面。

<span id="p2.12">![demo_new_micropost_3rd_edition](http://i.imgur.com/cBUsgoR.png)</span>  
图2.12 新微博建立页面（[/microposts/new](http://localhost:3000/microposts/new)）

接下来让我们继续前行，再创建一到两篇微博，注意其中一篇微博的用户id`user_id`应该为“1”，这是为了和之前[2.2.1](#2.2.1)中创建的用户一致。得到的结果应该和[图2.13](#p2.13)接近。

<span id="p2.13">![demo_micropost_index_3rd_edition](http://i.imgur.com/bLmgUrC.png)</span>  
图2.13 微博index页面（[/microposts](http://localhost:3000/microposts)）

###<span id="2.3.2">2.3.2 微博重在“微”</span>

微博，顾名思义，就是其内容应该不能太长。在Rails中使用**validation**功能就可以轻松实现这种限制；要使微博的长度限制在140个字以内（如同Twitter一样），我们使用长度验证（length validation）。现在，你需要打开文件`app/models/micropost.rb`，然后将[代码清单2.10](#l2.10)中的代码输入。

<code><span id="l2.10" style="font-size:12px; font-weight:bold; font-family:''">代码清单2.10：微博内容限制在140个字节以内。</span></code>   
`app/models/micropost.rb` 

```ruby
class Micropost < ActiveRecord::Base
  validates :content, length: { maximum: 140 }
end

```

[代码清单2.10](#l2.10)中的代码可能看起来让人有点迷惑，在[6.2](#6.2)我们有更多的内容来介绍“validation”验证功能，但是还有什么方法比直接测试更有效呢，让我们直接在微博页面上输入超过140个字节，看看有什么结果。就像[图2.14](#p2.14)所示，Rails错误信息提示微博的内容太长了。（我们将在[7.3.3](#7.3.3)了解到更多错误信息。）

<span id="p2.14">![micropost_length_error_3rd_edition](http://i.imgur.com/H9RoLMT.png)</span>  
图2.14 微博发布失败显示的错误信息

###<span id="2.3.3">2.3.3 用户有许多（has_many）微博</span>

Rails最强大的功能之一就是可以在不同数据模型之间建立**关联**（association）。在我们用户模型的例子中，一个用户可以有多篇微博。我们可以通过更新用户和微博模型的代码来实现这种关联，如下所示，用户模型[代码清单2.11](l2.11)和微博模型[代码清单2.12](l2.12)。

####代码清单2.11 单用户多微博  
`app/models/user.rb`

```ruby
class User < ActiveRecord::Base
  has_many :microposts
end

```

####代码清单2.12 微博关联到用户  
`app/models/micropost.rb`

```ruby
class Micropost < ActiveRecord::Base
  belongs_to :user
  validates :content, length: { maximum: 140 }
end

```

我们可以从[图2.15][#p2.15]中直观地看到关联结果。因为“microposts”表格中有“user_id”这一栏，所以Rails（通过使用Active Record）可以将微博和各个用户关联起来。

![micropost_user_association](http://i.imgur.com/8IAotYy.png)

图2.15 用户和微博之间的关联表格

在[11](#11)和[12](#12)中，我们将会把用户和微博关联起来，并将他们用类似Twitter网站的方式显示在页面上。现在，我们可以用控制台（console）来检查用户和微博的关联结果，控制台是Rails应用开发中非常强大的交互工具。首先我们在命令行中输入`rails console`来调出控制台（console），然后使用`User.first`命令从数据库中检索第一个用户（将检索到的内容放入变量“first_user”中）：

```
$ rails console
>> first_user = User.first
=> #<User id: 1, name: "Michael Hartl", email: "michael@example.org",
created_at: "2014-07-21 02:01:31", updated_at: "2014-07-21 02:01:31">
>> first_user.microposts
=> [#<Micropost id: 1, content: "First micropost!", user_id: 1, created_at:
"2014-07-21 02:37:37", updated_at: "2014-07-21 02:37:37">, #<Micropost id: 2,
content: "Second micropost", user_id: 1, created_at: "2014-07-21 02:38:54",
updated_at: "2014-07-21 02:38:54">]
>> micropost = first_user.microposts.first    # Micropost.first would also work.
=> #<Micropost id: 1, content: "First micropost!", user_id: 1, created_at:
"2014-07-21 02:37:37", updated_at: "2014-07-21 02:37:37">
>> micropost.user
=> #<User id: 1, name: "Michael Hartl", email: "michael@example.org",
created_at: "2014-07-21 02:01:31", updated_at: "2014-07-21 02:01:31">
>> exit

```

（在最后一行我加上了`exit`命令，这是用来退出控制台的命令。在其他系统中，你也可以使用“Ctrl+D”来退出）。现在我们已经使用代码`first_user.microposts`来访问“first_user”用户所有的微博。通过这个代码，Active Record会自动返回“user_id”和“first_user”的id一样的所有微博（在这个例子里，id="1"）。在[11](#11)和[12](#12)中我们将会学习更多关于Active Record中的这种关联工具。


###<span id="2.3.4">2.3.4 继承体系结构</span>

接下来，我们将简要介绍Rails的控制器和模型类继承，以此作为这章关于玩具网站学习的结尾。如果你对面向对象编程（object-oriented programming，简称OOP）有过相关经验，那我们这一节的讨论你将更容易理解；但是如果你没有学习过OOP，你可以随时跳过这一节。实际上，如果你对“类”（class）的概念不是很熟悉（[4.4](#4.4)会学习），我建议你到时候回过头来看看这一节。

我们从模型的继承开始入手。对比[代码清单2.13](l2.13)和[代码清单2.14](l2.14)的差异，我们发现，其实用户模型和微博模型都是从`ActiveRecord::Base`中继承而来（使用符号“<”表示继承），`ActiveRecord::Base`是由ActiveRecord提供的模型的基类；[图2.16][#p2.16]将这种关系用框图的形式表现出来。通过继承这个模型基类，我们的模型对象才能拥有与数据库通信，将数据库的列作为Ruby属性来处理的能力，还有其他很多功能，这里就不一一列举了。

####代码清单2.13 用户User类，高亮的地方表示继承  
`app/models/user.rb`

```ruby
class User < ActiveRecord::Base
  .
  .
  .
end

``` 

####代码清单2.14 微博Micropost类，高亮的地方表示继承  
`app/models/micropost.rb`

```ruby
class Micropost < ActiveRecord::Base
  .
  .
  .
end

``` 

![demo_model_inheritance](http://i.imgur.com/TJFM1rM.png)  
图2.16 用户模型和微博模型的继承体系结构

控制器的继承结构要稍微复杂一点。对比[代码清单2.15](l2.15)和[代码清单2.16](l2.16)的差异，我们可以看到用户控制器（User controller）和微博控制器（Microposts controller）都是从应用控制器（ApplicationController）继承来的。然后再看[代码清单2.17](l2.17)，我们将发现，其实**ApplicationController**也是从其他地方继承而来，具体来说就是从**ActionController::Base**继承来的；这是由Rails的Action Pack库提供的控制器基类。在[图2.17][#p2.17]的框图中显示了这些类之间的关系。

####代码清单2.15 用户控制器UsersController类，高亮的地方表示继承  
`app/controllers/users_controller.rb`

```ruby
class UsersController < ApplicationController
  .
  .
  .
end

``` 

####代码清单2.16 微博控制器MicropostsController类，高亮的地方表示继承  
`app/controllers/microposts_controller.rb`

```ruby
class UsersController < ApplicationController
  .
  .
  .
end

``` 

####代码清单2.17 ApplicationController类，高亮的地方表示继承  
`app/controllers/application_controller.rb`

```ruby
class ApplicationController < ActionController::Base
  .
  .
  .
end

``` 

![demo_controller_inheritance](http://i.imgur.com/4P5S2RK.png)  
图2.17 用户控制器和微博控制器的继承体系结构图

通过这种模型的继承，用户模型和微博模型可以从基类获得很多功能和属性（这个例子中的基类就是** ActionController::Base**），同时也包括对模型对象的操作，HTTP请求滤波，以及将视图渲染为HTML等等功能。由于Rails的所有控制器都从基类`ActionController`继承而来，因此在
应用控制器中定义的规则将自动适用于应用的每个动作。例如，在[8.4](#8.4)我们将看到如何引入辅助功能，为终极例程中的所有应用控制器添加登录和注销功能。

###<span id="2.3.5">2.3.5 部署toy app</span>

既然我们都已经完成了微博资源，现在是时候将代码推送到Bitbucket仓库中了：

```
$ git status
$ git add -A
$ git commit -m "Finish toy app"
$ git push

```

一般情况下，你需要养成“小规模改动，高频率提交”的代码管理习惯，但是本章为了方便给大家做演示，只需在现在，作为最后阶段一次性提交。

现在，你也可以将toy app部署到Heroku上，如同[1.5](#1.5)中的步骤：

```
$ git push heroku

```

（在这里我们假设你已经在[2.1](#2.1)中新建了一个Heroku应用，否则，你需要先运行`heroku create`命令，然后运行`git push heroku master`命令。）

为了使应用程序的数据库工作，你同时也需要迁移生产环境数据库：

```
$ heroku run rails db:migrate

```

这个命令将使用必要的用户和微博数据模型来更新Heroku的数据库。在运行迁移命令之后，你应该可以在生产环境中运行玩具网站了，同时这个应用是基于实际的PostgreSQL数据库，如[图2.18](#p2.18)所示。

![toy_app_production](http://i.imgur.com/VV4d45k.png)  
图2.18 在生产环境中运行toy app


##<span id="2.4">2.4 本章小结</span>

现在我们已经从一个终极高度来了解了Rails应用的结构。本章讲述的玩具网站有几个优点也有挺多缺点。

#####优点

* 全局视角来了解Rails
* 引出了MVC的概念
* 第一次了解REST架构（表现层状态转移）
* 开始接触数据模型
* 一个在生产环境中运行的基于数据库的web应用

#####缺点

* 没有用户自定义的布局和样式
* 没有静态页面（例如“首页”和“关于”）
* 没有用户密码
* 没有用户头像
* 没有登录界面
* 没有安全系统
* 没有自动化的用户/微博关联功能
* 没有“关注”和“被关注”功能
* 没有微博列表
* 没有实际的测试
* **没有真正理解开发过程**

本书的后续将专注于扬长避短。

<span id="2.4.1">2.4.1 本章我们学到了什么</span>

* 实用脚手架自动生成数据模型的代码，并通过页面和应用交互
* 脚手架适合快速上手，但不适合深入了解web开发
* Rails实用MVC架构（Model-View-Controller）开发web应用
* 由Rails我们了解到，为了和数据模型交互，REST架构定义了一个标准的URL动作和控制器动作
* Rails支持数据校验，在数据模型属性里可以添加约束条件
* Rails内含定义不同的数据模型之间的关联关系的功能
* 使用Rails控制台可以使用命令行的方式与应用进行交互

##<span id="2.5">2.5 本章练习题</span>

注意：练习题答案手册包含了本书的所有练习题详细解答，当购买本书时会免费赠送。详情请查看官网www.railstutorial.org。

1. [代码清单2.18](l2.18)用来为当前微博内容添加一个有效性验证功能，以此来保证该微博不是空的，是有内容的有效微博。确保能实现[如图2.19][#p2.19]显示的功能。
2. 更新[代码清单2.19](l2.19)，将`FILL_IN`修改未合适的代码，用来验证当前用户模型中用户的名字和email地址是否已经存在？（[如图2.20][#p2.20]）


####代码清单2.18 验证当前微博是否有效的代码 
`app/models/micropost.rb`

```ruby
class Micropost < ActiveRecord::Base
  belongs_to :user
  validates :content, length: { maximum: 140 },
                      presence: true
end

``` 

![micropost_content_cant_be_blank](http://i.imgur.com/8f1wri2.png)

图2.19 微博有效性验证效果图


####代码清单2.19 给用户模型添加存在验证 
`app/models/user.rb`

```ruby
class User < ActiveRecord::Base
  has_many :microposts
  validates FILL_IN, presence: true
  validates FILL_IN, presence: true
end

``` 

![user_presence_validations](http://i.imgur.com/mp3xjCs.png)  
图2.20 用户模型存在性检测效果图
















