## 第三章
#主要是静态的页面

这章，我们将开始开发专业级的示例程序，这个程序将贯穿本教程剩余的章节。尽管示例应用有用户、微博以及完整的登陆和授权框架，我们从一个好像非常有限的主题开始：创建静态页面。尽管它是显而易见的简单，制作静态页面是一个高建设度的练习，内涵丰富--对我们初生的应用程序来说是一个完美的开始。

尽管Rails被设计成制作数据库支持的动态网站，它在制作原始的HTML文件的静态页面也是非常出色的。实际上，使用Rails甚至对静态页面的生成一个显著的优势：我们能容易第添加一小部分动态的内容。在这章我们将学习怎样添加动态内容。沿着这个思路，我们将首次尝尝自动测试的味道，这将有助于增强我们对自己的代码的信心。而且，有一个好的测试将允许我们自信地重构代码，在不改变功能的情况下改变代码。

### 3.1 示例程序安装
和第二章一样，在我们开始前我们需要创建一个新的Rails工程，这次叫做**sample_app**，如同列表3.1所示。假如列表3.1的命令返回类似“Could not find 'railties”错误，这意味着你没有安装正确的Rails版本，你应该好好第检查一下是不是完全按照列表1.1的命令操作。
```
代码清单 3.1: 生成新的sample app
$ cd ~/workspace
$ rails _4.2.2_ new sample_app
$ cd sample_app/
```

（和章节2.1一样，云IDE的读者可以在和前两章同样的工作空间创建工程。重新创建一个新的工作空间是没有必要的）

和章节2.1一样，我们下一步是使用文本编辑器更新Gemfile，将我们需要的gem加入我们的应用程序里。列表3.2**test**组里的gem是和列表1.5、列表2.1完全一样的，它们对于安装可选的高级测试是需要的（章节3.7）。记住：假如你愿意为示例程序安装所有的gem，你这次应该使用列表11.67的代码。

```
代码清单 3.2: A Gemfile for the sample app.
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

group :test do
  gem 'minitest-reporters', '1.0.5'
  gem 'mini_backtrace',     '0.1.3'
  gem 'guard-minitest',     '2.3.1'
end

group :production do
  gem 'pg',             '0.17.1'
  gem 'rails_12factor', '0.0.2'
end
```
和前两章一样，我们运行**bundle install**安装Gemfile里面指定的gem，使用**--without production**跳过生产环境才需要安装的gem.

```
$ bundle install --without production
```
这样就在开发环境跳过了PostSQL使用的*pg* gem，在开发和测试环境使用SQLite。Heroku不推荐在开发环境和生产环境使用不同的数据库，但是对于我们这个示例程序来说没什么不同，而且SQLite与PostSQL相比，在本地安装和配置非常容易。万一你之前安装过gem的其他版本而不是**Gemfile**指定的版本，用**bundle update**命令*更新*gem是个不错的主意：
```
$ bundle update
```
做完这个，剩下的就是初始化Git仓库：
```
$ git init
$ git add -A
$ git commit -m "Initialize repository"
```
和第一个应用程序一样，我建议更新一下**README**文件（位于应用程序的根目录），让它变得更加有帮助和更具描述性。添加列表3.3的内容：
```
列表 3.3: 为sample app修饰过的README文件
# Ruby on Rails Tutorial: sample application

This is the sample application for the
[*Ruby on Rails Tutorial:
Learn Web Development with Rails*](http://www.railstutorial.org/)
by [Michael Hartl](http://www.michaelhartl.com/).
```
最后，提交变化
```
$ git commit -am "Improve the README"
```
你可以回忆一下在1.4.4我们使用Git命令**git commit -a -m "Message"**，用标记（-a）表示“所有变化”，（-m）表示信息。如同在第二个命令显示的那样，Git也允许我们把两个标记合在一起使用**git commit -am "Message"**。

因为我们将在本书剩余的部分都使用这个例子，因此在[Bitbucket创建一个新的仓库](https://bitbucket.org/repo/create)并且推送至远程仓库是个好主意：
```
$ git remote add origin git@bitbucket.org:<username>/sample_app.git
$ git push -u origin --all #首次推送仓库和它的参考
```
为了避免后续集成时令人头痛的问题，在初期就将应用部署至Heroku也是个好想法。如同在第一章和第二章一样，我建议跟随列表1.8和1.9里的“hello, world!”完成。然后提交变化，推送至Heroku：
```
$ git commit -am "Add hell"
$ heroku create
git push heroku master
```
(如同在1.5节一样，你可能会看见一些警告信息，你现在应该忽视它们。我们将在7.5节消除它们）除了Heroku应用的地址，结果应该和图1.18一样。

当你进一步深入本书剩余的部分，我建议你规律地推送和部署应用程序，它会自动远程备份，让你尽可能早地发现程序错误。假如你在Heroku上碰到问题，确定看一看产品日志尽力诊断问题：
```
$ heroku logs
```
注：假如你要把真实的应用程序部署至Heroku，请确定按照章节7.5描述的那样配置你的Web服务器。

## 3.2 静态页面
随着在3.1里的所有准备结束，我们准备开始开发示例应用程序。在这节，首先通过创建一套Rails动作（actions）和仅包含静态HTML的视图（views）的动态页面迈出我们的第一步。Rails动作把里面的控制器（controllers，就是1.3.3节中介绍的MVC中的C）联系在一起，包含常用目的的一套动作。我们在第二章领略了控制器，一旦我们更全面地揭开REST架构（从第6章开始）的神秘面纱，将会有一个更加深入的理解。为了找到感觉，回忆一下在1.3节（图1.4）介绍的Rails的目录结构会对我们有帮助的。在这节，我们将开始主要在**app/controllers**和**# app/views**目录工作。

回忆一下1.4.4节，当我们使用Git时，在一个单独的主题分支修改代码比起直接在主分支修改来说是一个更好的软件开发实践。假如你正用Git做版本控制，你应该运行以下命令，为静态页面检出（checkout）主题分支：
```
$ git checkout master
$ git checkout -b static-pages
```
(这里第一行只是确定你从主分支开始，以便**static-pages**主题分支是基于主分支的。假如你已经在主分支了，你可以跳过第一条命令。)

### 3.2.1 生成静态页面
从静态页面开始，和第二章一样的，我们同样使用Rails的**generate**生成控制器的命令来生成脚手架。因为我们要开始处理静态页面了，所以我们就叫它静态页面控制器，使用[驼峰式命名法](https://en.wikipedia.org/wiki/CamelCase)命名为**StaticPages**。我们也计划添加一个主页，一个帮助页面，和一个关于页面，使用小写的动作名称**home、help、about**。**generate**脚本以动作名列表作为参数，所以我们将主页、帮助页面直接加在命令行上，故意不把“关于”页面的动作名加上，以便在后面看看怎么添加（3.3节）。命令的结果是生成如代码清单3.4所示的静态页面控制器。
```
代码清单 3.4: Generating a Static Pages controller.
$ rails generate controller StaticPages home help
      create  app/controllers/static_pages_controller.rb
       route  get 'static_pages/help'
       route  get 'static_pages/home'
      invoke  erb
      create    # app/views/static_pages
      create    # app/views/static_pages/home.html.erb
      create    # app/views/static_pages/help.html.erb
      invoke  test_unit
      create    test/controllers/static_pages_controller_test.rb
      invoke  helper
      create    app/helpers/static_pages_helper.rb
      invoke    test_unit
      create      test/helpers/static_pages_helper_test.rb
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/static_pages.coffee
      invoke    css
      create      app/assets/stylesheets/static_pages.css
```
顺便提一下，虽然不值得一提，**rails g**是**rails generate**的缩写，是Rails支持的几个缩写之一（表3.1）。声明一下，本教程总是使用全拼的命令，但是在真实的开发过程中，Rails开发者常常使用表3.1所示的一个或多个缩写。

Full command	    | Shortcut
-----------------|---------
$ rails server	  | $ rails s
$ rails console	| $ rails c
$ rails generate	| $ rails g
$ bundle install	| $ bundle
$ rails test	      | $ rake

【表3.1：一些Rails命令的缩写

在继续之前，假如你正使用Git，把我们的静态控制器加入到远程仓库是个很好的主意。
```
$ git status
$ git add -A
$ git commit -m "添加一个静态页面控制器"
$ git push -u origin static-pages
```
最后一行命令把**static-pages**主题分支推送至Bitbucket。
随后推送时可以忽略别的参数，只写
```
$ git push
```
提交和推送是我平常在真实开发过程中遵循的模式，但是为了简化，从现在开始我会忽略类似这种中间提交。

在代码清单3.4里，记住我们已经按照驼峰命名法传递过控制器的名字，它会创建一个[蛇形命名法](https://en.wikipedia.org/wiki/Snake_case)的控制器文件，所以StaticPages控制器生成文件名为**static_pages_controller.rb**的文件。这只是个惯例，实际上使用蛇形命名法也是一样的：命令
```
$ rails generate controller static_pages ...
```
也是生成**static_pages_controller.rb**文件。因为Ruby用驼峰命名法为类命名（章节4.4），我的偏好是使用控制器的驼峰命名法，但是这只是个人习惯问题。（因为Ruby文件名一贯使用蛇形命名法，Rails生成器会使用**underscore**将驼峰命名法的单词转为蛇形命名法。）

顺便提一下，假如你生成代码时犯了错误，知道怎么取消操作是很有用的。关于Rails里是怎么取消操作的，请参考注3.1。
    
		注3.1 取消操作
		即使你非常小心，当你开发Rails应用程序时你也搞错一些事情。令人高兴的是，Rails有一些帮你恢复的工具。
		
		一个常见的场景是想要还原生成的代码--例如，当你想改变你之前生成的控制器的名称，先要删除已经生成的文件。因为Rails和控制器一道创建了数量可观的辅助文件（如代码清单3.4所示），
		这不是和移除控制器文件本身一样容易；取消生成文件意味着取消主要的文件，而且也包括辅助的文件。（实际上，正如我们再章节2.2和2.3中看到的一样，
		**rails generate**可以自动编辑**routes.rb**文件，这个我们也需要自动还原。）在Rails里，可以通过rails destroy加上生成的要素的名称
		完成。在这里，这两个命令取消了彼此的输出：
		```
		$ rails generate controller StaticPages home help
		$ rails destroy  controller StaticPages home help
		```
		类似地，在第6章，我们按照如下方式生成模型**model**：
		```
		$ rails generate model User name:string email:string
		```
		可以使用以下命令还原
		```
		$ rails destroy model User
		```
		(在这个例子里，证明了我们可以省略其他命令行参数。等到了第6章，看看你是不是能弄明白为什么。)
		另一个和模型相关的技术是还原迁移（migrations），我们在第2章简单的介绍过，在第6章将会经常用到。迁移使用以下命令改变了数据的状态
		```
		$ bundle exec rails db:migrate
		```
		我们可以使用以下命令还原一步
		```
		$ bundle exec rails db:rollback
		```
		还原到初始状态，我们可以用
		```
		$ bundle exec rails db:migrate VERSION=0
		```
		正如你所猜到的，用其他数字代替0会迁移到数字对应的迁移，版本数字来自原列在迁移后面的数字。
		
		掌握了这些技术，我们已经准备好了应对开发过程中的不可避免的混乱局面了。

为了理解这个页面来自于哪里，让我们打开文本编辑器，看看静态页面控制器。应该看上去和代码清单3.6一样。你可能注意到了，和第2章用户和微博控制器不一样，静态页面控制器没有使用标准的REST动作。这对于一个静态页面的集合是正常的：REST架构不适用于每个问题。

```
代码清单 3.6: 代码清单3.4生成的静态页面控制器
# app/controllers/static_pages_controller.rb
 class StaticPagesController < ApplicationController

  def home
  end

  def help
  end
end
```

我们看见在static_pages_controller.rb文件里，如代码清单3.6所示，使用class定义了一个类（class），在这里，这个类名为**StaticPagesController**。类是一种组织函数（也叫方法）很方便的途径，就像用**def**定义的home和help动作一样。正如2.3.4节讨论的那样，尖括号**<**表示StaticPagesController继承于Rails的ApplicationController类；就像我们将要看见的一样，这意味着我们的页面已经具有了大量的Rails指定的功能。（我们将在章节4.4中学习更多类和继承的知识。）

在这个静态页面控制器的例子里，它的两个方法均是空的：
```ruby
def home
end

def help
end
```
在Ruby中，这些方法什么也不做。在Rails中，情况却步相同。因为StaticPagesController是一个Ruby类，但是因为它继承于ApplicationController，所以这些方法的行为对于Rails来说是有特定意义的：当访问URL /static_pages/home, Rails在静态页面控制类里查找和执行home动作，然后渲染视图（1.3.3节中MVC中的V），回应这个动作。在目前的情况，home动作是空的，所以所有访问/static_pages/home所做得就是渲染视图。那么视图是什么样的，怎么找到它？

假如你再看看代码清单3.4里的输出，你可能可以猜出来在动作和视图之间的响应：像home动作有一个对应的视图，叫做home.html.erb。我们将在章节3.4里学习到**.erb**的意思；**.html**你可能不惊奇，因为看起来像HTML(代码清单3.7)。
```
代码清单 3.7: 为Home页面生成的视图
# app/views/static_pages/home.html.erb
 <h1>StaticPages#home</h1>
<p>Find me in # app/views/static_pages/home.html.erb</p>
```

**help**动作的视图类似（代码清单3.8）
```
代码清单 3.8: 为Help页面生成的视图
# app/views/static_pages/help.html.erb
 <h1>StaticPages#help</h1>
<p>Find me in # app/views/static_pages/help.html.erb</p>
```

这些视图只是占位符：它们有个一级的标题，（在标签**h1**里）和一段（标签**p**）包含对应文件完整路径的段落。

## 3.2.2 自定义静态页面
在3.4节我们开始加入一点点动态的内容，但是如代码清单3.7和3.8说明的一样，强调了一点：Rails视图可以仅包含静态的HTML文件。这意味着即使我们没有Rails的相关知识，我们可以开代码清单始自定义主页和帮助页面，如同代码清单3.9和3.10所示。
```
代码清单 3.9: 为Home页面自定义HTML。
# app/views/static_pages/home.html.erb
 <h1>Sample App</h1>
<p>
  This is the home page for the
  <a href="http://www.railstutorial.org/">Ruby on Rails Tutorial</a>
  sample application.
</p>
```
```
代码清单 3.10: 为Help页面自定义HTML。
# app/views/static_pages/help.html.erb
 <h1>Help</h1>
<p>
  Get help on the Ruby on Rails Tutorial at the
  <a href="http://www.railstutorial.org/#help">Rails Tutorial help section</a>.
  To get help on this sample app, see the
  <a href="http://www.railstutorial.org/book"><em>Ruby on Rails Tutorial</em>
  book</a>.
</p>
```
代码清单3.9和3.10的结果如图3.3和3.4.
![图3.3：自定义主页](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/custom_home_page.png)
![图3.4：自定义帮助y](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/custom_help_page_3rd_edition.png)

## 3.3 从测试开始
既然我们已经创建并且为我们的示例程序的主页和帮助页面添加了内容（3.2.2节），现在我们准备再添加一个关于页面。当我们进行这一类的变化，好的开发实践是先写自动化测试来验证是否正确地实现了这个特性。在创建应用程序的过程中，测试套件就像安全网，像一套可执行的文档。当这些都正确实施时，即便编写测试要写一些额外的代码，也可以加快我们开发的速度，因为我们追踪bug的时间变少了。一旦我们擅长写测试时这就是真理，尽可能早地开始实践是很重要的，这也是其中的原因之一。

尽管所有的Rails开发者都同意测试是好主意，但是在细节方面也有不少分歧。关于测试驱动开发（TDD）有一个尤其活跃的争论，TDD是一种测试技术，程序员先写失败的测试，然后在编写能让测试通过的代码。本教程采取轻量级的、简便的方法去测试，当需要的时候才使用测试驱动开发（TDD），而不是像教条主义一样的（注3.3）。


    <h3><b>注3.3 何时需要测试</b></h3>
		当决定何时和怎样测试时，理解为什么测试是很有帮助的。在我来看，写自动测试有三个主要的好处：
		  1. 测试可以有效防止功能特性因为某些原因不工作而造成开发回溯。
		  2. 测试允许重构代码（例如，不改变功能的情况下重构），而且更有信心。
		  3. 测试同时扮演应用程序客户的角色，因此有助于决定系统设计以及系统其他部分的接口。
			
		尽管以上的好处没有一个是要求先写测试的，有许多情况测试驱动开发（TDD）是你工具包里有价值的工具之一。决定何时以及怎样测试部分依赖于你写的测试是多么舒服;许多开发者发现当他们越擅长写测试，他们越倾向于先写测试。同时，它也依赖于和应用程序代码相关的测试编写难度有多大、想要的特性是否清楚，而且未来这个特性被更改的可能性有多大。

    在这篇教程里，有一套关于何时我们应该先写测试（或者根本不写测试）的指导标准是有帮助的。基于我个人的经验，这里我有一些建议：
    * 当测试非常短或者和它测试的应用代码相比很简单，倾向于先写测试。
    * 当想要的行为还不是特别清楚，倾向于先写代码，然后针对结果写测试。
    * 因为安全是顶级优先的，安全模型的错误测试先写。
    * 无论何时发现bug，写一个重现bug的测试，优先防止开发回溯，然后编写补丁。
    * 未来可能改变的（例如HTML结构的细节）倾向于不写测试。
    * 重构代码前先写测试，专注于可能导致程序崩溃的测试。

    在实践中，以上的指导思路意味着我们通常先写控制器和模型测试，集成测试（测试连贯模型、视图和控制器的）次之，而且当我们正在编写的代码是不太容易崩溃的、或者不太容易出错的应用程序代码、或者可能会变化的（常发生在视图）代码时，我们常常跳过测试。


我们主要的测试工具将是控制器测试（从本节开始）、模型测试（从第6章开始），还有集成测试（从第7章开始）。集成测试尤其强大，因为它允许我们模拟用户使用网页浏览器和我们的应用程序进行交互。集成测试最终是我们主要的测试技术，但是控制器测试会让我们从容易的地方着手。

### 3.3.1 我们的第一个测试
现在是给我们的应用程序添加关于页面的的时候了。正如我们将看见的一样，这个测试很简单，所以我们根据注3.3的指导方针先写测试，然后我们使用失败的测试来驱动我们编写应用程序代码。

从测试开始有点挑战性，涉及到大量的Rails和Ruby知识。在我们目前的阶段，写测试可能让人感到歇斯底里的胆怯。幸运的是，Rails已经帮我们完成了最艰难的部分，因为**rails generate controller**（代码清单3.4）自动生成了测试文件，我们可以从这里开始：
```
$ ls test/controllers/
static_pages_controller_test.rb
```

让我们看看它（代码清单3.11）
```
代码清单 3.11: The default tests for the StaticPages controller. 绿色
# test/controllers/static_pages_controller_test.rb
 require 'test_helper'

class StaticPagesControllerTest < ActionDispatch::IntegrationTest
  test "should get home" do
    get static_pages_home_url
    assert_response :success
  end

  test "should get help" do
    get static_pages_help_url
    assert_response :success
  end
end
```

现在理解代码清单3.11里面的语法细节还不是很重要，但是我们看到在测试文件中有两个测试，对应着我们在代码清单3.4的命令行上的每个控制器动作。每个测试只是简简单单得到一个动作，然后确认（通过断言）结果成功。这里的**get**的用法表示我们的测试想要主页和帮助也面是普通的网页，使用GET请求进入（旁注3.2）。这个响应**:success**是一个抽象的HTTP[状态码](http://en.wikipedia.org/wiki/List_of_HTTP_status_codes)(在这里，是[200 OK](http://en.wikipedia.org/wiki/List_of_HTTP_status_codes#2xx_Success)。换句话说，像这个测试：
```ruby
 test "should get home" do
    get static_pages_home_url
    assert_response :success
  end
```
是说“让我们通过发送GET请求到**home**动作来测试主页，然后确认我们收到了“成功”状态码。”

为了开始我们的TDD测试驱动开发循环，我们需要运行我们的测试套件来确认目前的测试通过。我们用如下命令，使用**rails**（旁注2.1）来测试。
```
代码清单 3.12：GREEN
$ bundle exec rails test
2 tests, 2 assertions, 0 failures, 0 errors, 0 skips
```
正如我们想要的，我们的测试套件开始时候是通过的（绿的）。（除非你根据3.7.1节添加了mini-test-reporter，你不会真的看见绿色）。顺便提一下，刚开始测试时需要花费一点时间，这是因为两个因素：（1）启动Spring服务时需要预加载部分Rails环境，这个只会在第一次启动Spring时发生；（2）和Ruby启动时间相关的一些因素。（第二个因素在使用3.7.3节建议的Guard时会改善）

###3.3.2 红色
如同在旁注3.3里说明的一样，测试驱动开发需要先写测试，再写能让测试通过的应用程序代码，然后如果必要的话重构代码。因为许多测试工具用红色表示测试失败，绿色表示测试通过，所以就有了“红色，绿色，重构”更替。在本节中，我们先完成这个循环的第一步，先写变红的（失败的）测试。然后我们在3.3.3节中让测试变绿（通过），然后在3.4.3节中重构。

我们的第一步是写一个关于页面的失败测试。遵循代码清单3.11里的模型，你可能大概猜到怎么写测试了，具体如代码清单3.13.
```
代码清单 3.13: 为“关于”页面写的测试。红色
# test/controllers/static_pages_controller_test.rb

require 'test_helper'

class StaticPagesControllerTest < ActionDispatch::IntegrationTest
  test "should get home" do
    get static_pages_home_url
    assert_response :success
  end

  test "should get help" do
    get static_pages_help_url
    assert_response :success
  end

  test "shold get about" do
    get statice_pages_about_url
    assert_response :success
  end
end
```

我们从代码清单3.13中看到的高亮的代码就是关于页面的的测试。除了用“about”代替了“home”和“help”之外，基本上是一模一样的。

正如我们想要的，测试开始时失败了：

```ruby
代码清单3.14：红色
$ bundle exec rails test
3 tests, 2 assertions, 0 failures, 1 errors, 0 skips
```
### 3.3.3 绿色
既然我们有了一个失败的测试（红色），我们将使用这个失败测试的错误信息指导我们通过测试（绿色），因此实现一个工作的关于页面。

我们从检查失败测试输出的错误信息开始：
```terminal
代码清单 3.15：红色
$ bundle exec rails test
Error:
StaticPagesControllerTest#test_shold_get_about:
NameError: undefined local variable or method `statice_pages_about_url' for
#<StaticPagesControllerTest:0x007fd5fede8fb0>
    test/controllers/static_pages_controller_test.rb:15:in `block in
    <class:StaticPagesControllerTest>'

# bin/rails test test/controllers/static_pages_controller_test.rb:14
           '
```
这里的错误信息是说未定义名为“static_pages_about_url”本地变量或者方法，这暗示我们需要给路由文件添加一行代码。我们依据代码清单3.5的模式完成它，如同代码清单3.16所示。
```ruby
代码清单3.16：添加关于页面的路由。红色
# config/routes.rb
Rails.application.routes.draw do
  get 'static_pages/home'
  get 'static_pages/help'
  get 'static_pages/about'
  .
  .
  .
end
```
在代码清单3.16里显示的高亮的行告诉Rails路由URL/static_pages/about的GET请求到StaticPagesController的about动作。

再次运行测试，我们看见还是红色的，但是现在错误信息已经改变了：
```terminal
代码清单3.17：红色
$ bundle exec rails test
AbstractController::ActionNotFound:
The action 'about' could not be found for StaticPagesController
```
错误信息提示在StaticPagesController里找不到**about**动作，我们可以照着代码清单3.6中**home**和**help**代码添加如代码清单3.18里一样的代码。
```ruby
代码清单3.18：在StaticPagesController里添加about动作。红色
class StaticPagesController < ApplicationController

  def home
  end

  def help
  end

  def about
  end
end
```
这次我们的测试通过了。
```terminal
$ bundle exec rails test
...
Finished in 0.380464s, 7.8851 runs/s, 7.8851 assertions/s.
3 runs, 3 assertions, 0 failures, 0 errors, 0 skips
```
但是，当我们运行服务器，然后访问about页面时，在命令行会提示：
```
$ rails server
=> Booting Puma
=> Rails 5.0.0.beta1 application starting in development on
http://localhost:3000
=> Run `rails server -h` for more startup options
=> Ctrl-C to shutdown server

# 访问http://localhost:3000/about时
Started GET "/static_pages/about" for ::1 at 2015-12-23 17:11:32 +0800
Processing by StaticPagesController#about as HTML
No template found for StaticPagesController#about, rendering head :no_content
Completed 204 No Content in 11ms (ActiveRecord: 0.0ms)
```
服务器返回204状态码，这表示返回的HTML文件没有内容。而在终端里输出的提示说明缺少模板，在Rails的中，模板和视图是一回事。如同3.2.1节里面所描述的一样，**home**动作是和名为**home.html.erb**的视图相联系的，这个文件位于**# app/views/static_pages**目录，这意味着我们需要在同一个目录创建一个名为**about.html.erb**的文件。

创建一个文件根据系统有所不同，但是大多数文本编辑器会让你所在的目录按control-点击，然后弹出一个有“新文件”的菜单。你也可以使用“文件”菜单新建文件，然后选择正确的位置保存它。最后，你可以使用你最喜欢的小技巧，使用Unix的**touch**命令：
```terminal
$ touch # app/views/static_pages/about.html.erb
```
尽管**touch**被用来设计成更新文件或目录的时间戳而不影响其他方面，副作用是假如不存在，就会创建一个新的文件。（假如使用云IDE，按照1.3.1节描述的方法刷新一下目录树。）

一旦你在正确的目录创建了**about.html.erb**文件以后，把代码清单3.19的代码复制进去。
```html
代码清单3.19: 关于页面的代码。绿色
# # app/views/static_pages/about.html.erb
 <h1>About</h1>
<p>
  The <a href="http://www.railstutorial.org/"><em>Ruby on Rails
  Tutorial</em></a> is a
  <a href="http://www.railstutorial.org/book">book</a> and
  <a href="http://screencasts.railstutorial.org/">screencast series</a>
  to teach web development with
  <a href="http://rubyonrails.org/">Ruby on Rails</a>.
  This is the sample application for the tutorial.
</p>
```

当然，在浏览器里看看效果，确定我们的测试不是完全“疯了”,这从来不会是一个坏主意（图3.5）。

![图3.5]：新关于页面（/static_pages/about）。

### 3.3.4 重构
既然我们的测试已经变绿了，我们可以带着自信重构我们的代码了。当开发应用程序的时候，刚开始写的代码常常很“臭”，意思是丑陋，臃肿，或者充满了重复。计算机不管代码看上去怎样，当然，但是人类会，所以通过频繁的重构保持代码清洁是很重要的。尽管我们现在的示例程序对于重构来说太小了，但是代码的臭味从每个缝隙渗透出来，所以我们将从3.4.3节开始重构。

## 3.4 略微动态的网页

既然我们为几个静态页面创建了动作和视图，我们可以通过添加一些改变每个页面基础的内容让它们略微动态起来：我们将通过改变页面的标题反应它的内容。改变标题是不是真的动态内容，这点是有争议的，但是不管怎样，这为我们第7章毫不含糊地动态内容打下了基础。

我们的计划是编辑主页、帮助页和关于页面来改变每页的标题，也就是在我们的页面视图里使用**\<title>**标签
。大部分的浏览器在浏览器的顶端显示标题标签里面的内容，它对搜索引擎优化也是重要的。我们将使用完整的“
红色，绿色，重构”循环：首先通过为我们的页面标题添加简单的测试（红色），然后给三个页面添加标题（绿色
），最后使用**layout**文件来消除重复（重构）。在本节结束，我们的三个静态页面有类似“<页面名称>| Ruby
 on Rails Tutorial Sample App”，这里标题的第一部分将根据页面变化（表3.2）。

**rails new**命令（代码清单3.1）创建了一个默认的布局文件，但是我们先忽略它，这个对我们学习是有益的，
我们先重命名该文件：
```terminal
$ mv # app/views/layouts/application.html.erb layout_file #temporary change
```
你平常不会在真正的应用程序里做这样的事，但是假如我们现在先不用它，我们就会更加容易地理解布局文件的用途。

页面 | URL | 基础标题 | 可变标题
-----|-----|----------|------
主页 |/static_pages/home| "Ruby on Rails Tutorial Sample App" | "Home"
帮助 |/static_pages/help| "Ruby on Rails Turorial Sample App" | "Help"
关于| /static_pages/about| "Ruby on Rails Tutorial Sample App" |"About"

表3.2：示例应用（基本都是）静态的页面

### 3.4.1 测试标题（红色）
为了添加页面标题，我们需要学习（或复习）一个典型的网页的结构，如代码清单3.21所示。
```
代码清单3.21: 典型的网页的HTML结构
<!DOCTYPE html>
<html>
  <head>
    <title>Greeting</title>
  </head>
  <body>
    <p>Hello, world!</p>
  </body>
</html>
```
代码清单3.21里的结构包含**文档类型**，或**doctype**，在浏览器顶部的声明告诉浏览器我们使用的是那个版本的HTML（在这个例子里是HTML5)；**head**部分，在这个例子里是**title**标签里面的“Greeting”；和**body**部分，这里是指**p**标签里的“Hello， world！”。（缩进是可选的--HTML对空白字符不敏感，忽略制表格和空格--但是这让文档的结构更清楚）

我们将为表3.2里的每个标题写一个简单的测试，合并代码清单3.13里的测试和**assert_select**方法，让我们测试某个HTML标签是不是存在（有时叫“选择器”,因此取了这个名字）：
```ruby
assert_select "title", "Home | Ruby on Rails Tutorial Sample App"
```
以上代码检查包含 "Home | Ruby on Rails Tutorial Sample App"字符串的\<title>标签是否存在。把这个思路应用到其他三个的静态页面，代码清单3.22显示了测试代码。

```ruby
代码清单 3.22: 静态页面控制器标题测试。红色
# test/controllers/static_pages_controller_test.rb
require 'test_helper'

class StaticPagesControllerTest < ActionDispatch::IntegrationTest

  test "should get home" do
    get static_pages_home_url
    assert_response :success
    assert_select "title", "Home | Ruby on Rails Tutorial Sample App"
  end

  test "should get help" do
    get static_pages_help_url
    assert_response :success
    assert_select "title", "Help | Ruby on Rails Tutorial Sample App"
  end

  test "should get about" do
    get static_pages_about_url
    assert_response :success
    assert_select "title", "About | Ruby on Rails Tutorial Sample App"
  end
end
```
（假如你觉得基础标题“Ruby on Rails Tutorial Sample App”啰嗦，看看3.6节的练习。）

代码清单3.22写好后，你应该验证一下测试集是红色的：
```
代码清单 3.23: 红色
$ bundle exec rails test
3 tests, 6 assertions, 3 failures, 0 errors, 0 skips
```

### 3.4.2 添加页面标题（绿色）
现在我们将在每个页面增加一个标题，让3.4.1节缩写的测试通过。把代码清单3.21里基本的HTML结构从代码清单3.9生成代码清单3.24.

```
代码清单 3.24: Home页面完整的HTML文件。 红色
# # app/views/static_pages/home.html.erb
 <!DOCTYPE html>
<html>
  <head>
    <title>Home | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>Sample App</h1>
    <p>
      This is the home page for the
      <a href="http://www.railstutorial.org/">Ruby on Rails Tutorial</a>
      sample application.
    </p>
  </body>
</html>
```
对应的网页显示如图3.6
![图3.6: 带标题的主页](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/home_view_full_html.png)

参考这个模型，把帮助页面（代码清单3.10）和关于页面（代码清单3.19）改为代码清单3.25和代码清单3.26.

```
代码清单 3.25: 完整的Help页面HTML文件。 红色
# app/views/static_pages/help.html.erb
 <!DOCTYPE html>
<html>
  <head>
    <title>Help | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>Help</h1>
    <p>
      Get help on the Ruby on Rails Tutorial at the
      <a href="http://www.railstutorial.org/#help">Rails Tutorial help
      section</a>.
      To get help on this sample app, see the
      <a href="http://www.railstutorial.org/book"><em>Ruby on Rails
      Tutorial</em> book</a>.
    </p>
  </body>
</html>
```
```
代码清单 3.26: 完整的About页面HTML文件。 绿色
# app/views/static_pages/about.html.erb
 <!DOCTYPE html>
<html>
  <head>
    <title>About | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>About</h1>
    <p>
      The <a href="http://www.railstutorial.org/"><em>Ruby on Rails
      Tutorial</em></a> is a
      <a href="http://www.railstutorial.org/book">book</a> and
      <a href="http://screencasts.railstutorial.org/">screencast series</a>
      to teach web development with
      <a href="http://rubyonrails.org/">Ruby on Rails</a>.
      This is the sample application for the tutorial.
    </p>
  </body>
</html>
```
到这一步，测试集应该变回绿色：
```
代码清单 3.27: 绿色
$ bundle exec rails test
3 tests, 6 assertions, 0 failures, 0 errors, 0 skips
```
###3.4.3 布局和内嵌Ruby（重构）
在这节，我们已经取得了许多成就，生成了三个使用Rails控制器和动作的有用的页面，但是他们是纯粹静态的HTML，因此没有显示出Rails得强大。而且，它们三个页面重复的可怕：
* 页面标题几乎一模一样
* “Ruby on Rails Tutorial Sample App"在三个标题里都有
* 整个HTML框架在每个页面都重复了

这些重复的代码打破了“不要重复自己”(DRY，英语“Don't Repeat Yourself”的缩写)原则；在这节，我们将移除重复的代码，让我们的代码遵循DRY原则。最后，我们从3.4.2节开始重新运行测试确认标题正确。

矛盾地，我们先通过增加一些代码来消除重复：我们将创建页面标题，和当前的非常像是，完全匹配。这将使得一下子移除所有重复的代码变得更加容易。

这个技术将会在视图内使用内嵌的Ruby。因为主页、帮助页面、关于页面的标题有一个可变的部分，我们将使用一个Rails特有的函数，叫做**provide**来在每个页面上设置不同的标题。我们看看文件**home.html.erb**里的代码（代码清单3.28)，通过替换**home.html.erb**视图文件中的“Home”实现了。
```
代码清单 3.28: 带嵌入代码的Home页面视图。绿色
# # app/views/static_pages/home.html.erb
 <% provide(:title, "Home") %>
<!DOCTYPE html>
<html>
  <head>
    <title><%= yield(:title) %> | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>Sample App</h1>
    <p>
      This is the home page for the
      <a href="http://www.railstutorial.org/">Ruby on Rails Tutorial</a>
      sample application.
    </p>
  </body>
</html>
```

代码清单3.28是我们的第一个内嵌Ruby的例子，也叫ERB。（现在你知道为什么HTML视图文件的后缀是**.html.erb**.)ERb是网页中包含动态内容主要的模板系统。代码
```
<% provide(:title, "Home") %>
使用<% ... %>Rails会调用**provide**函数，将字符串“Home”和标签**:title**联系在一起。然后，在标题中，我们使用类似的相关的标记<%= ... %>使用Ruby's**yield**函数插入模板中的标题：

```erb
<title><%= yield(:title) %> | Ruby on Rails Tutorial Sample App</title>
```
(两个内嵌Ruby标记的区别是<% ... %>执行里面的代码， 然而<%= ...
%>执行代码，然后将结果插入模板。）页面的显示和之前的一模一样，只是现在标题可变的部分是由ERB动态生成的。

我们可以通过运行3.4.2节中的测试，确认所有的这些仍是绿色的：

```
代码清单 3.29: 绿色
$ bundle exec rails test
3 tests, 6 assertions, 0 failures, 0 errors, 0 skips
```

然后我们替换帮助页面和关于页面相应的代码（代码清单3.30和3.31）。

```
代码清单 3.30: 嵌入Ruby代码的Help页面。绿色
# # app/views/static_pages/help.html.erb
<% provide(:title, "Help") %>
<!DOCTYPE html>
<html>
  <head>
    <title><%= yield(:title) %> | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>Help</h1>
    <p>
      Get help on the Ruby on Rails Tutorial at the
      <a href="http://www.railstutorial.org/#help">Rails Tutorial help
      section</a>.
      To get help on this sample app, see the
      <a href="http://www.railstutorial.org/book"><em>Ruby on Rails
      Tutorial</em> book</a>.
    </p>
  </body>
</html>
```

```
代码清单 3.31: 嵌入Ruby代码的About页面。绿色
# # app/views/static_pages/about.html.erb
 <% provide(:title, "About") %>
<!DOCTYPE html>
<html>
  <head>
    <title><%= yield(:title) %> | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>About</h1>
    <p>
      The <a href="http://www.railstutorial.org/"><em>Ruby on Rails
      Tutorial</em></a> is a
      <a href="http://www.railstutorial.org/book">book</a> and
      <a href="http://screencasts.railstutorial.org/">screencast series</a>
      to teach web development with
      <a href="http://rubyonrails.org/">Ruby on Rails</a>.
      This is the sample application for the tutorial.
    </p>
  </body>
</html>
```
既然我们已经用ERB替换了页面标题的可变部分，我们每个页面看起来像：
```
<% provide(:title, "The Title") %>
<!DOCTYPE html>
<html>
  <head>
    <title><%= yield(:title) %> | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    Contents
  </body>
</html>
```
换句话说，所有页面在结构上是相同的，包括标题标签的内容，除了**body**标签里的内容不同。

为了提炼出一致的结构，Rails自带一个特殊的布局文件，**application.html.erb**，我们在这节（3.4节）开始的时候重命名的那个文件，现在我们还原回去：
```terminal
$ mv layout_file # app/views/layouts/application.html.erb
```

为了让布局工作，我们不得不用内嵌Ruby代码代替默认的标题：
```ruby
<title><%= yield(:title) %> | Ruby on Rails Tutorial Sample App</title>
```

修改后的布局文件如代码清单3.32所示。
```
代码清单 3.32: 示例网站的布局（layout）文件。绿色
# # app/views/layouts/application.html.erb
 <!DOCTYPE html>
<html>
  <head>
    <title><%= yield(:title) %> | Ruby on Rails Tutorial Sample App</title>
    <%= stylesheet_link_tag    'application', media: 'all',
                                              'data-turbolinks-track' => true %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
    <%= csrf_meta_tags %>
  </head>
  <body>
    <%= yield %>
  </body>
</html>
```
注意这里特殊的一行
```ruby
<%= yield %>
```
这行代码是为了把每页的内容插入布局文件。现在想要确切地知道这是怎么回事还不重要；重要的是使用这个布局文件，确保，例如，访问网页/static_pages/home将**home.html.erb**转换成HTML文件，然后替换<%= yield %>。

默认的Rails布局文件也包含没有什么使用价值的几个文件：

```html
<%= stylesheet_link_tag ... %>
<%= javascript_include_tag "application", ... %>
<%= csrf_meta_tags %>
```

这些代码用来将应用程序中的样式文件（CSS）和Javascript文件包含进来，它们是asset
pipeline（5.2.1节）的一部分，**csrf_meta_tags**方法是防止[跨页面劫持](http://en.wikipedia.org/wiki/Cross-site_request_forgery)(CSRF)，一种恶意的网络攻击。

当然，代码清单3.28、3.30和3.31中的视图现在还都是包含布局的HTML结构，所以我们不得不移除它，仅仅留下内部的内容。清理后的视图如代码清单3.33，3.34和3.35一样。
```erb
代码清单 3.33: 移除HTML结构的Home视图。 绿色
# app/views/static_pages/home.html.erb
 <% provide(:title, "Home") %>
<h1>Sample App</h1>
<p>
  This is the home page for the
  <a href="http://www.railstutorial.org/">Ruby on Rails Tutorial</a>
  sample application.
</p>
```
```erb
代码清单 3.34: 移除HTML结构的Help视图。 绿色
# app/views/static_pages/help.html.erb
 <% provide(:title, "Help") %>
<h1>Help</h1>
<p>
  Get help on the Ruby on Rails Tutorial at the
  <a href="http://www.railstutorial.org/#help">Rails Tutorial help section</a>.
  To get help on this sample app, see the
  <a href="http://www.railstutorial.org/book"><em>Ruby on Rails Tutorial</em>
  book</a>.
</p>
```

```
代码清单 3.35: 移除HTML结构的About视图。 绿色
# app/views/static_pages/about.html.erb
 <% provide(:title, "About") %>
<h1>About</h1>
<p>
  The <a href="http://www.railstutorial.org/"><em>Ruby on Rails
  Tutorial</em></a> is a
  <a href="http://www.railstutorial.org/book">book</a> and
  <a href="http://screencasts.railstutorial.org/">screencast series</a>
  to teach web development with
  <a href="http://rubyonrails.org/">Ruby on Rails</a>.
  This is the sample application for the tutorial.
</p>
```
随着这些视图修改完毕，主页、帮助页面和关于页面和之前的一样，但是只有少量的代码是重复的。

经验显示即使相当简单的重构也容易犯错，也能轻易地偏离正确方向。这是为什么一个好的测试集是非常有价值的原因之一。不断重复地检查每个页面--早期还不难，但是随着项目的快速增长就会变得不好操作--我们可以简单的通过测试集确认仍然是绿色的：
```
代码清单 3.36: 绿色
$ bundle exec rails test
3 tests, 6 assertions, 0 failures, 0 errors, 0 skips
```

这不能证明我们的代码百分百正确，但是它大大地增加了它正确的可能性。因此它为我们提供了一个安全网，保护我们未来远离bug。

### 3.4.4 设置根路由
既然我们已经自定义了我们网站的页面，在测试集方面有了一个好的开始，在我们继续之前，让我们设置一下应用程序的根路由。正如1.3.4节和2.2.2节中描述的那样，需要修改**routes.rb**文件，连接到我们选择的页面，这里我们选择主页。（到这步，我也推荐从AppicationController中移除**hello**动作，假如你在章节3.1添加过的话）如同代码清单3.37，这意味着用以下代码替换代码清单3.5生成的**get**规则：
```
root 'static_pages#home'
```

这改变了URL**static_pages/home**到“控制/动作对”static_pages#home的路由，确保了对“/”的GET请求路由到静态页面控制器的**home**动作。路由文件最后的结果如图3.7.（记住，代码清单3.37里，先前的路由**static_pages/home**将不再工作）
```
代码清单 3.37: 把根路由映射到static_pages/home页面。
# config/routes.rb
 Rails.application.routes.draw do
  root 'static_pages#home'
  get  'static_pages/help'
  get  'static_pages/about'
end
```

![图3.7: 根路由主页](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/home_root_route.png)

## 3.5 结论
从浏览器的效果来看，这章几乎没有完成任何事情：我们从静态页面开始，然后用大部分都是静态的页面结束。但是表面是具有欺骗性的：通过用Rails控制器、动作和视图，该是我们添加任意数量的动态内容到我们网站的时候了。逐渐揭开这些神秘面纱，是本教程剩余章节的任务。

在我们继续前，让我们花几分钟提交我们主题分支的变化，然后把它们合并到主分支。回到3.2节中，为了创建静态页面，我们创建了一个Git分支。假如在我们一直开发的过程当中你没有提交，先提交一下，表示我们到达了一个停止点：

```
$ git add -A
$ git commit -m "Finish static pages"
```

然后使用和1.4.4节一样的技术，把变化合并到主分支。

```
$ git checkout master
$ git merge static-pages
```

一旦你到达了像这样的一个停止点，推送你的代码到远程仓库通常是一个好的主意（假如你按照1.4.3节的步骤，就是指Bitbucket）：
```
$ git push
```

我也推荐你将程序部署至Heroku：

```
$ bundle exec rails test
$ git push heroku
```

这里我们要在部署前先完整运行一下我们的测试集，确保它完全通过，这也是开发的一个好习惯。

### 3.5.1 本章我们学到了什么
* 第三次，我们经历了从零开始创建了一个新的Rails应用，安装了必要的gem包，推送到远程仓库，最后部署到生产。
* **rails**脚本使用**rails generate controller 控制器名称 <可选的动作名称参数>**生成了一个新的控制器。
* 在文件**config/routes.rb**里定义了一个新路由。
* Rails视图可以包含静态HTML或者内嵌Ruby（ERb）
* 自动测试允许我们写测试集，驱动新特性的开发，允许有信心的重构，捕捉回溯。
* 使用“红色，绿色，重构”循环测试驱动开发。
* 因为Rails布局允许我们在应用程序里使用普通模板，所以消灭了重复。

## 3.6 练习
说明：练习答案手册，包括本书中每个练习的解决方案，购买本书可以免费从[www.railstutorial.org]网站获取。

从现在开始直到本教程结束，我推荐在一个单独的主题分支解决练习：
```
$ git checkout static-pages
$ git checkout -b static-pages-exercises
```
这样就会防止和教程出现冲突。

一旦你对自己的解决方案感到满意，你可以把练习的主题分支推送到远程仓库（假如你已经设置了）：

```terminal
<解决第一个练习>
$ git commit -am "取消重复（解决练习3.1)"
<解决第二个练习>
$ git add -A
$ git commit -m "添加一个联系页面（解决练习3.2)"
$ git push -u origin static-pages-exercises
$ git checkout master
```
(作为为将来的开发准备，这里最后一步检出主分支，但是我们为了避免和教程剩余章节冲突，我们不会把变化合并至主分支。）在以后的章节中，分支和提交信息将会有所不同，当然，但是基本思路是一样的，
1. 你可能已经注意到在静态页面控制器测试（代码清单3.22）里有一些重复。具体来说，基本的标题，“Ruby
on Rails Tutorial Sample App”，每个测试里有一样。使用特殊的函数**setup**，这个会在每次测试之前都自动运行，确认代码清单3.38里的测试仍是绿色的。（代码清单3.38使用一个*实例变量*，在2.2.2节中快速的看见过，在4.4.5节里会覆盖到，和插入字符串一起，都将在4.2.2节中介绍）

2. 为示例程序添加一个联系页面。模仿代码清单3.13，先写为了网页访问URL /static_pages/contact通过测试写一个测试标题“Contact | Ruby on Rails Tutorial Sample App”的测试。通过和在代码清单3.3.3中创建关于页面一样的步骤，包括添加代码清单3.39里的内容。（注意，为了保持练习独立，代码清单3.39不会合并代码清单3.38里的代码。）

```
代码清单 3.38: 有基本标题的静态页面控制器测试。 绿色
# test/controllers/static_pages_controller_test.rb
 require 'test_helper'

class StaticPagesControllerTest < ActionDispatch::IntegrationTest

  def setup
    @base_title = "Ruby on Rails Tutorial Sample App"
  end

  test "should get home" do
    get static_pages_home_url
    assert_response :success
    assert_select "title", "Home | #{@base_title}"
  end

  test "should get help" do
    get static_pages_help_url
    assert_response :success
    assert_select "title", "Help | #{@base_title}"
  end

  test "should get about" do
    get static_pages_about_url
    assert_response :success
    assert_select "title", "About | #{@base_title}"
  end
end
```

```
代码清单 3.39：Contact页面的内容。
# app/views/static_pages/contact.html.erb
 <% provide(:title, "Contact") %>
<h1>Contact</h1>
<p>
  Contact the Ruby on Rails Tutorial about the sample app at the
  <a href="http://www.railstutorial.org/#contact">contact page</a>.
</p>
```

## 3.7 高级测试安装
这节是选修的，描述在本书配套的视频资源里使用的测试环境的安装。有三个注意的要素：一个是加强的通过/失败报告者（3.7.1节），还有一个是过滤失败测试回溯的工具（3.7.2节），和一个自动化测试运行器，监测文件变化，自动运行相应的测试（3.7.3节）。这节中的代码比较高级，仅仅是为了方便才在这里出现，目前还不要求你能理解他。

**这节里的变化，应合并进主分支。**
```
$ git checkout master
```
### 3.7.1 迷你测试报告者
为了默认的Rails测试在合适的时候显示红色和绿色,我推荐把代码清单3.40的代码加到你的测试帮助文件，从而可以使用**[minitest-reporters](https://github.com/kern/minitest-reporters)**gem。

```
代码清单 3.40: Configuring the tests to show 红色 and 绿色.
# test/test_helper.rb
 ENV['RAILS_ENV'] ||= 'test'
require File.expand_path('../../config/environment', __FILE__)
require 'rails/test_help'
require "minitest/reporters"
Minitest::Reporters.use!

class ActiveSupport::TestCase
  # Setup all fixtures in test/fixtures/*.yml for all tests in alphabetical
  # order.
  fixtures :all

  # Add more helper methods to be used by all tests here...
end
```
如在云IDE里的显示（图3.8），出现了从红色到绿色的变化。

！[图3.8：云IDE显示红色和绿色](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd0_edition/images/figures/红色_to_绿色.png)

### 3.7.2 回溯静默
当遇见错误或者失败测试，测试运行者显示“堆栈追踪”或“追踪回溯”，从应用程序里追踪失败的原因。这个追踪回溯对于查找问题来说非常有用，在某些系统（包括云IDE）会通过应用程序代码进入各种gem依赖，包括Rails自己。回溯的结果常常太长，尤其因为问题原因通常是由程序的代码引起，而不是它的依赖引起的时候。

解决方案是过滤追踪信息消除不想要的信息。这需要**[mini_backtrace](https://github.com/metaskills/mini_backtrace)**，包含在代码清单3.2里，和*backtrace silencer*一起使用。在云IDE，多数不需要的信息包含字符串*rvm*（Ruby版本管理器），所以我推荐如代码清单3.41里显示的静默者把它们过滤掉。

```
代码清单 3.41: 为RVM添加backtrace silencer。
# config/initializers/backtrace_silencers.rb
 # Be sure to restart your server when you modify this file.

# You can add backtrace silencers for libraries that you're using but don't
# wish to see in your backtraces.
Rails.backtrace_cleaner.add_silencer { |line| line =~ /rvm/ }

# You can also remove all the silencers if you're trying to debug a problem
# that might stem from framework code.
# Rails.backtrace_cleaner.remove_silencers!
```

如同在代码清单3.41里的一句评论显示，你应该在添加了静默者之后重启本地服务器。

### 3.7.3 使用Guard自动测试
使用**rails test**命令有一个令人心烦的方面就是不得不切换到命令行，然后手动运行测试。为了避免这种不方便，我们可以使用Guard来自动运行测试。Guard监视文件系统的变化，例如，当我们改变了文件**static_pages_controller_test.rb**,仅仅这些测试运行。甚至更好，我们可以配置以便当**home.html.erb**改变了，**static_pages_controller_test.rb**自动运行。

在代码清单3.2的Gemfile里一句包含了*guard* gem， 所以我们只需要初始化它：
```
$ bundle exec guard init
Writing new Guardfile to /home/ubuntu/workspace/sample_app/Guardfile
00:51:32 - INFO - minitest guard added to Guardfile, feel free to edit it
```

然后我们编辑**Guardfile**以便Guard当集成测试和视图更新时运行正确的测试（代码清单3.42）。（由于文件较长，而且属于高级特性，我推荐拷贝黏贴就好了）

```
代码清单 3.42: A custom Guardfile.
# Defines the matching rules for Guard.
guard :minitest, spring: true, all_on_start: false do
  watch(%r{^test/(.*)/?(.*)_test\.rb$})
  watch('test/test_helper.rb') { 'test' }
  watch('config/routes.rb')    { integration_tests }
  watch(%r{^app/models/(.*?)\.rb$}) do |matches|
    "test/models/#{matches[1]}_test.rb"
  end
  watch(%r{^app/controllers/(.*?)_controller\.rb$}) do |matches|
    resource_tests(matches[1])
  end
  watch(%r{^# app/views/([^/]*?)/.*\.html\.erb$}) do |matches|
    ["test/controllers/#{matches[1]}_controller_test.rb"] +
    integration_tests(matches[1])
  end
  watch(%r{^app/helpers/(.*?)_helper\.rb$}) do |matches|
    integration_tests(matches[1])
  end
  watch('app/views/layouts/application.html.erb') do
    'test/integration/site_layout_test.rb'
  end
  watch('app/helpers/sessions_helper.rb') do
    integration_tests << 'test/helpers/sessions_helper_test.rb'
  end
  watch('app/controllers/sessions_controller.rb') do
    ['test/controllers/sessions_controller_test.rb',
     'test/integration/users_login_test.rb']
  end
  watch('app/controllers/account_activations_controller.rb') do
    'test/integration/users_signup_test.rb'
  end
  watch(%r{# app/views/users/*}) do
    resource_tests('users') +
    ['test/integration/microposts_interface_test.rb']
  end
end

# Returns the integration tests corresponding to the given resource.
def integration_tests(resource = :all)
  if resource == :all
    Dir["test/integration/*"]
  else
    Dir["test/integration/#{resource}_*.rb"]
  end
end

# Returns the controller tests corresponding to the given resource.
def controller_test(resource)
  "test/controllers/#{resource}_controller_test.rb"
end

# Returns all tests for the given resource.
def resource_tests(resource)
  integration_tests(resource) << controller_test(resource)
end
```
这里
```
guard :minitest, spring:true, all_on_start:false do
```
让Guard使用Rails提供的Spring服务加快加载时间，然而也阻止Guard重新开始运行完整测试
为了防止Spring和Git使用Guard引起的冲突，你应该将**spring/目录加入到**.gitignore**中，当Git添加文件s时忽略它们。在云IDE里：
1.点击右边的齿轮（图3.9）
2.选择“显示隐藏文件在应用程序的根目录”显示**.gitignore**文件（图3.10）
3.双击**.gitignore**文件（图3.11）来打开它，然后加入代码清单3.43的内容。

![图3.9: 在文件导航面板的齿轮图标](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/file_navigator_gear_icon.png)
![图3.10：在文件导航栏显示隐藏文件](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/show_hidden_files.png)
![图3.11：平常隐藏的**.gitignore**文件可见了](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/gitignore.png)
```
代码清单 3.43: 把Spring添加到.gitignore文件中。
# See https://help.github.com/articles/ignoring-files for more about ignoring
# files.
#
# If you find yourself ignoring temporary files generated by your text editor
# or operating system, you probably want to add a global ignore instead:
#   git config --global core.excludesfile '~/.gitignore_global'

# Ignore bundler config.
/.bundle

# Ignore the default SQLite database.
/db/*.sqlite3
/db/*.sqlite3-journƒal

# Ignore all logfiles and tempfiles.
/log/*.log
/tmp

# Ignore Spring files.
/spring/*.pid
```
Spring服务仍然还有点难搞，有时Spring进场将累积，拖慢你的测试的表现。假如你的测试变的不同寻常的慢，检查系统进程，假如需要的话杀死进程（注：3.4）。

    注3.4 Unix进程
    在类Unix系统，例如Linux和OS X，用户和系统任务都在一个定义的很好的，叫做进程的容器里运行。为了看见你的系统所有进程，你可以使用**ps**命令和**aux**参数：
    $ ps aux
    为了依据类型过滤进程，你可以运行**ps**通过**grep**模式匹配器使用Unix管道|：
    $ ps aux | grep spring
      ubuntu 12241 0.3 0.5 589960 178416 ? Ssl Sep20 1:46
      spring app | sample_app | started 7 hours ago
    结果显示进程的细节，但是最重要事情是第一个数字，它是进程ID，或者pid。为了消除不想要的进程，使用kill命令发送Unix杀死代码（正好是9）到那个pid：
```
$ kill -9 12241
```
这是我推荐的杀死单个进程的技术，例如缓慢的Rails服务器（通过命令*ps aux | grep
server),但是有时杀死匹配某个模式的所有进程，例如当你想用杀死所有的*spring*进程。在这种情况，你应该首先试试用spring命令终止：
```
$ sping stop
```
有时这不起作用，尽管，你能用以下命令啥事所有名为spring的进程：
```
$ pkill -9 -f spring
```
任何时候有些进程行为和预料的不一样，或者冻结了，运行ps
aux命令看看什么情况，然后运行kill -9 <pid>或者pkill -9 -f <名字>清洁进程。

一旦Guard配置好，你应该新开一个终端（如果1.3.2节中介绍的一样）运行以下命令：

```
$ bundle exec guard
```

这个代码清单3.42的规则是为本教程优化，当控制器的代码改变后自动运行（例如）集成测试。为了运行所有测试，在**guard>**点击回车键。（有时可能会给出一个和Spring相关的错误提示。解决这个问题，就是再点击一次回车键）

按Ctrl-D退出Guard。为了给Guard添加额外的匹配规则，参考代码清单3.42的例子，Guard的[读我](https://github.com/guard/guard)，和Guard的[wiki](https://github.com/guard/guard/wiki).

