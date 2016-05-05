#1 从零到部署

欢迎来到《Ruby on Rails教程：使用Rails进行Web开发》。这本教程的目的是教你快速开发网站，我们选择的开发工具是最流行的Ruby on Rails web应用开发框架。如果你是一个新手，《Ruby on Rails 教程》将会给你一个完整的关于网页应用开发流程的详细介绍，其中包括一些基础内容：如Ruby，Rails，HTML和CSS，数据库，代码版本控制，代码测试，以及网站部署等等。这些内容足以让你成为一名专业的程序员或者开创一家互联网企业。除此之外，如果你已经了解网站的开发过程，这本教程可以快速地教会你Rails架构的精髓，包括MVC、REST、代码生成、数据库迁移、路由，以及在HTML文件中内嵌Ruby代码等。无论你是哪种类型的读者，当你完成本书的示例和练习后，你将会拥有很好的基础，能够更容易地从繁荣的编程教育生态系统，如更加高阶的书籍、博客、以及视频中吸收更多的专业知识。

本书以集成的方法构建三个难度逐渐递增的网站。首先是从最小的“hello”网站开始（章节1.3），然后是功能稍微强大一些的“toy（玩具）”网站（章节2），接下来就是一个货真价实的微博网站（章节3到章节12）。如同这些网页程序的名字所表现出来的字面上的意思，本书中这些应用程序的开发并不针对任何特定类型的网站；尽管本书最后的一个网站与一个有名的微博网站很相似（巧合的是这个网站最初也是用Rails架构编写的），但贯穿于本书的重点是一些Web应用开发的基本原则。因此，你一旦你掌握了这些基础知识，无论你以后想开发哪种类型的网站，都会得心应手。

当你想通过本书学会开发网站时，必须具备哪些背景知识呢？关于这个问题，我们会在章节1.1.1里做一次深入的探讨。网站开发是一门很有挑战的学科，尤其是对于初学者来说。虽然本书最初设定的目标读者群体是那些拥有一定的编程经验，或者网页开发经验的人，但是实际上我发现纯粹的初学者在读者群中也占有很大的比重。作为对广大初学者读者的回报，这本教程通过几个重要的步骤来降低初学者学习Rails的门槛（如注1.1）。


    注 1.1 降低学习门槛

    第三版着眼于降低学习Rails的入门门槛，主要通过采取以下措施
    
    * 使用标准的云开发环境IDE（章节1.2），避开了许多初学者在安装、配置新的开发环境过程中碰到的各种各样的问题
    * 使用Rails开发的“默认知识栈”，包括内建的MiniTest测试框架
    * 删减了很多外部依赖库， 如RSpec,Cucumber,Capybara,Factory Girl
    * 一个轻量级但是灵活度很高的测试方法
    * 推迟或者删减了Spork,RubyTest等需要复杂配置的选项
    * 弱化了特定版本的Rails的一些特殊功能，更强调一些网站开发通用的原则

    我希望通过这些在第三版的Ruby on Rails教程所作的改变，能够使本书的读者群更广泛一些。



在第一章，我们从安装一些必要的软件，以及配置我们的开发环境（1.2）来开始Ruby on Rails之旅。随后我们创建第一个Rails应用程序，名字叫做**hello_app**。Ruby on Rails教程注重于体现一种良好的软件开发实践，因此当我们创建Rails应用程序后，立即用Git来对我们的应用程序进行版本控制，把这个网站源程序放入到 Git代码仓库（1.4）。在这一章我们还要把我们的第一个应用程序发布到网上供大家浏览（1.5）。

在第二章，我们会创建第二个网站，这个网站的目的在于演示Rails应用程序的一些基本工作原理。为了快速上手，我们将会通过Rails的脚手架（scaffold）来快速生成代码，创建一个名为“toy_app”的网站。因为这些使用脚手架生成的代码不够优雅而且还很复杂，所以2这章的重点主要放在如何通过应用程序的网址（URI，一般称为URL）来进行交互。这些通过网页浏览器就可以实现。

本书的剩下的几章（第3章到第12章）将集中开发一个真实的网站，名字叫做“sample_app”。我们将从零开始手动编写所有的代码。我们将使用一些工具组合来开发这个范例网站：草图、测试驱动开发（TDD），以及集成测试。在第三章中我们将创建一些静态的网页，然后添加一点动态的内容。在第四章我们将学习一些开发Rails程序用到的Ruby语言的语法知识。然后，从第五章到第十章，我们通过创建网站骨架来实现网站应用程序的基础结构：用户数据模型、完整的注册和认证系统（包括账号激活和密码重置）。最后，在第十一章和十二章我们会添加一个微博平台以及一些社交的特性，实现一个可良好运行的范例网站。


    注 1.2 脚手架：快速，简单，有趣

    从Rails一露面，其显而易见的特性便令无数开发者激动。这要从由Rails的创建者David Heinemeier Hansson制作的那个著名的“15分钟博客视频”说起。这个视频和后续同类视频非常好的阐述了Rails的强大之处，我建议大家去看看。但是需要特别说明的一点是：他们之所以可以完成那15分钟的炫技表演，是因为使用了Rails的脚手架（scaffolding）特性，这个特性重在使用代码生成工具，犹如魔术棒一样的Rails命令**generate scaffold**来实现。
		
    当编写Ruby on Rails教程时，我试图通过脚手架（scaffiold）方法实现——因为它快速，简单，非常有诱惑力。但是其代码的复杂度和学习曲线的陡峭度会把一个Rails初学者彻底搞糊涂。你可能学会了怎么使用，但是却看不明白。完全遵循手脚架的方法，可能把你变成一个脚本生成的行家里手，但对Rails的原理一无所知。
		
    因此，在本书中，我们使用几乎完全相反的方法：虽然在第二章中我们会使用手脚架来开发小型的“toy app”网站，但是本书的核心内容是从第三章引入的sample app网站应用程序开始。在开发范例程序的的每个阶段，我们将会编写非常少的，少到几乎一口就能吃掉的代码。这些代码简单到一看就明白，但是也非常新颖，对初学者来说也不乏挑战。在这种逐步累积的过程中，你会逐渐对Rails有更深入的、灵活的认识。一旦你牢固掌握了这些基础，无论以后进行哪种类型的网页应用程序开发，都可以游刃有余。

##1.1 简介
Ruby on Rails（简称“Rails”）是使用Ruby语言编写的网站开发架构。自从2004年问世以来，Ruby on Rails快速地成为了最强大的、最流行的动态网页应用开发工具之一。各种不同类型的网站，例如Airbnb，Basecamp，Disney，GitHub，Hulu，Kickstarter，Shopify，Twitter，以及Yellow Pages等都是采用Rails开发的。还有其他很多网站开发分包商都专注于Rails开发服务，例如ENTP，Thoughtbot，Pivotal Labs，Hashrocket，以及HappyFunCorp，加上无数独立顾问、培训师、应用开发承包商等，都在使用Rails。

是什么因素使得Rails独领风骚？首先，Ruby on Rails是100%开源的，遵循MIT许可证协议，因此他可以不受任何限制地下载或使用。Rails的成功还源于他优雅而紧凑的设计风格；通过Ruby语言极强的可扩展性，Rails高效地创建了DSL（域名专属语言），结果是许多普通的网页开发任务，例如生成HTML，创建数据模型，以及URL路由，都变得很容易，并且最终的应用程序代码也很简洁，可读性很强。

同时Rails也能够快速地适应新的网页应用开发技术和架构设计。例如，Rails架构是第一批完全吸纳和实现REST风格的网页应用开发架构（我们将会在本书的后续章节来学习）。当其他架构成功地开发出新的技术后，Rails的作者David Heinemeier Hanssson以及Rails的核心团队就会毫不犹豫地将那些好的技术想法吸收到Rails架构中。可能最有代表性的例子就是Rails架构和Merb架构的融合，其中Merb也是使用Ruby语言的编写一个Web框架，应该说它是Rails Web框架的竞争对手，但是现在Rails吸收了Merb架构的模块化设计，稳定的API，拥有了更加优良的性能。

最后，Rails还从很多异常热心的、多样化的团体那里得到很多帮助和支持。包括成千上百个开源代码的贡献者、高出席率的研讨会、大量的gem（针对特定问题，例如分页和图片上传等的解决方法库）、内容丰富多彩的博客以及众多的论坛和IRC频道。庞大的Rails程序员团体也让处理不可避免的程序错误变得更容易：“谷歌一下错误信息”的方法常常会让你搜到许多相关问题的博客或者论坛文章。

###1.1.1 预备知识

阅读本书之前基本不需要任何的背景知识——本书不仅包含了Rails的教程，同时包含了Ruby语言（Rails使用Ruby语言编写）的语法介绍、默认的Rails测试架构程序（MiniTest）、Unix命令行、HTML、CSS、一小部分的JaveScript，以及一点SQL（数据库查询语言）等内容的介绍。虽然有很多的知识需要吸收消化，然而我一般只推荐读者在阅读本书之前有一些HTML的基础知识以及一点编程经验即可。这么说吧，大量的初学者已经选择使用本书的前两个版本来学习网页编程，所以就算你仅有一点点相关经验，我也建议你可以尝试一下。如果你感到学习过程举步维艰，你可以随时回头去补充你需要的知识，这些知识可以从下面的资源清单上可以找到。另外一个推荐的方法是这本Rails教程的读者们推荐的，就是连续读两次这本教程；第一次读完以后你可能会为你学到的东西感到吃惊，而阅读第二次时你就会发现这些知识对你来说是多么容易。

一个常见的问题是在我们学习Rails之前是否需要先学习Ruby。这个问题的答案取决于你个人的学习习惯和你之前有多少编程经验。如果你是那种更倾向于系统化的学习方法，喜欢从基础学起，或者之前从未有过编程经验的话，那么从Ruby编程语言开始学起对你来说会是不错的选择。在这个情况下，我推荐Chris Pine的[《Learn to Program》](http://pragprog.com/book/ltp2/learn-to-program)和Peter Cooper的[《Beginning Ruby》](http://www.amazon.com/gp/product/1430223634)。有些刚开始用Rails开发的初学者一上手就热衷于制作网页应用程序，在开始写网页时都不愿意先完整地学习一遍关于Ruby的内容。在这种情况下，我建议在开始阅读本书之前，通过一个简短的交互式Ruby学习教程[《Try Ruby》](http://tryruby.org/)来对Ruby有一个大致的认识。如果你仍然觉得本书比较难以理解，你可能可以尝试一下Daniel Kehoe的[《Learn Ruby on Rails》](http://learn-rails.com/learn-ruby-on-rails.html)或者[《One Month Rails》](http://mbsy.co/7Zdc7)，这两个教程可能比本书更适合纯粹的初学者学习。


在本书的最后，不论你从何处开始学习，你都应该好好留意下面列出的从中级到高级的Rails相关资源。以下是我特别推荐的：

 * [Code School](http://mbsy.co/6VQ8l): 拥有非常棒的交互式在线编程课程
 * [The Turing School of Software & Design](http://turing.io/friends/tutorial): 一个全日制的27周培训课程，在科罗拉多州丹佛市，如果使用“codeRAILSTUTORIAL500”，本书的读者将会有500美金的折扣
 * [Tealeaf Academy](http://www.gotealeaf.com/railstutorial)：一个很好的网上Rails开发训练营（包括一些高级资料）
 * [Thinkful](http://www.thinkful.com/a/railstutorial)：一个在线教学课程，由一个有经验丰富的工程师带你学习，以项目为基础的教学课程；
 * [Pragmatic Studio](https://pragmaticstudio.com/refs/railstutorial)：由Mike和Nicole Clark创办的在线教学课程。以及《Programming Ruby》的作者Dave Thomas，Mike执教Rails的第一课，这个课程我在2006年也教过。
 * [RailsCasts](http://railscasts.com/)：由Ryan Bates创办的非常棒的Rails视频演示（大部分是免费的）；
 * [RailsApps](https://tutorials.railsapps.org/hartl)：拥有海量的特定主题的Rails项目和教程；
 * [Rails Guides](http://guides.rubyonrails.org/)：最新的Rails参考文档，也是Ruby on Rails的官方文档

###1.1.2 本书常用约定

本书中大部分的用词惯例都是一看就明白的，即你可以从字面意思上了解他的意义。在这一节，我会列出一些例外情形。

本书中很多例子都用到了命令行的命令。简单来说，所有的命令行范例都使用了Unix风格的命令行提示符（美元符号），如下所示：

```
$ echo "hello, world"
hello, world

```

如同在章节1.2中提到的，我建议使用不同类别的操作系统的用户（尤其是Windows用户）都使用我们提供的云IDE（1.2.1），本身就具有内嵌的Unix(Linux)的命令行。这尤其有用，因为Rails附带了很多能在命令行直接运行的命令。例如，在1.3.2，我们将会运行一个本地的网络开发服务器，使用“rails server”命令：

`$ rails server`

紧跟着命令行提示符，本书也将使用Unix风格惯例的路径分隔符（斜杠"/"）。例如，Sample应用程序的**production.rb**配置文件的路径表示如下：

`config/environments/production.rb`

这个文件的路径应该理解为以应用程序的目录为根目录的相对路径，因为这个路径可能会因操作系统而异；在云IDE上（章节1.2.1），路径看起来就像：
`/home/ubuntu/workspace/sample_app/`

因此“production.rb”的完整路径如下：

`/home/ubuntu/workspace/sample_app/config/environments/production.rb`

为了书写的方便和简洁，我会省略应用程序的路径，只写成
`config/evironments/production.rb`

本书会经常展示从不同程序（shell命令，版本控制状态，Ruby程序等）输出的结果。由于不同的电脑系统间会有很多细微的差别，因此输出的结果可能不是完全相同，不要担心。另外，有些命令在某些系统中可能会报错；与其尝试通读本书列出的各种错误原因，我更倾向于“谷歌搜索一下错误信息”准则，这更接近于我们的实际软件开发方式。当你在阅读本书过程中遇到问题时，我建议从[Rails Tutorial help section](http://www.railstutorial.org/#help)所列的资源清单里寻求帮助。

因为本书涵盖了Rails应用测试工具，所以知道某段代码是否引起测试失败常常会很有帮助。当一个特定的程序块测试不通过时会被标记为红色，相反若通过测试时会被标记为绿色。为了方便，如果测试不通过则代码用红色表示，若测试通过则代码用绿色表示。

本书的每一章节都包含了一些练习题，读者可自行决定是否完成那些练习题，但是我建议最好是认真去完成它们。为了保持论述主题独立于练习题，一般情况下练习题的答案不会出现在后续的代码清单中。在某些特殊的情况下，练习题的答案会被后续的章节引用，他们会在需要的地方明确地给出答案。

最后，为了方便描述，本书约定了两种形式，以确保书中那么多的代码能够更容易理解。其一，某些代码行会包含多种高亮方式，如下所示：

```ruby 
class User < ActiveRecord::Base
  validates :name,  presence: true
  validates :email, presence: true
end

```

这种高亮方式是一种典型的方法，表示给出的例程中最重要的新代码，也常常用来表示当前代码行和之前的代码的区别之处（这种情况也不是很经常出现）。其二，为了叙述的简洁和方便，书中很多代码将会包含一些点，如下所示：

```ruby
class User < ActiveRecord::Base
  .
  .
  .
  has_secure_password
end
```

这些点代表省略的代码段，所以不能简单的复制这些代码，因为它们是不完整的，无法运行的。

##1.2 开发环境搭建及程序运行
就算是有经验的Rails开发者，在安装Ruby,Rails,还有其他相关软件的时候也会碰到很多问题。造成这个现象的原因是安装环境的多样性：不同的操作系统、不同的软件版本、文本编辑器以及集成开发环境（integrated development environment (缩写IDE)）偏好配置等。如果有人已经在自己的机器上把Rails的开发环境搭建好了，并且已有自己的配置的话，那是最理想的了，但是就像在注1.1所提到的一样，我们强烈建议新用户绕开安装和配置自己的计算机，使用我们提供的云IDE，这样就可以在最开始的时候避免那些繁琐的安装过程。由于云IDE可以运行在任何一个普通的浏览器上，因此说这个开发平台是可以运行在不同的系统环境下面的，尤其对于Windows操作系统用户来说更值得期待，因为在Windows操作系统上搭建Rails开发环境往往是一件很痛苦的事情。如果你不惧困难，想挑战一把的话，我推荐你可以到这个网站去寻找些帮助：[InstallRails.com](http://installrails.com/)。

###1.2.1 开发环境
每一个人都有自己的偏好和习惯，因此可以这么说，有多少个Rails程序员，就有多少种开发环境。为了避免这些复杂的问题，本书统一使用一款非常优秀的云IDE：[Cloud9](https://c9.io/)。需要特别说明的是，为了配合本书第三版的教学，我还特地和Cloud9合作，专门定制了适合本书的开发环境。在Cloud9定制了堪称专家级的Rails开发环境，平台上面的预配置以及所需的协同软件应有尽有，包括Ruby,RubyGems，Git。(如此看来Rails软件是唯一需要我们自己安装的，而且这是有意而为的（[1.2.2](#1.2.2)))。同时云IDE还包含了Web开发所需的三个组件：文本编辑器、文件导航栏、还有一个命令行终端（如图1.1）。在众多的功能中，文本编辑器有一个很实用的功能，那就是“在文件中查找”(Find in Files)的全局查找功能，我认为这个功能对于那种比较大型的Ruby或者Rails项目来说非常重要，因为代码量很大的时候，你就不知道有些函数在哪个文件里定义或者被哪个地方调用了。最后，如果你还是坚持要自己搭建Rails开发环境的话（当然我还是希望大家能够多学点其他工具），我还是要再次推销一下我们的云端开发环境，他可以给开发者提供功能强大的文本编辑器和其他开发工具。

<span id="p1.1">![The anatomy of the cloud IDE](http://i.imgur.com/v3FZr6l.png)</span>
图1.1 云端IDE界面


以下是云端开发环境的使用步骤：
   1. 注册一个免费的[Cloud9](https://c9.io/)账号
   2. 点击“Go to your Dashboard”(进入你的主控面板)
   3. 选择“Create New Workspace”(新建工作空间)
   4. 如图1.2所示，创建一个名为“rails-tutorial”的工作空间（不是带下划线的“rails_tutorial”），选择“Private to the people I invite”（仅对我邀请的人开放），然后选择下面的“Rails Tutorial”图标（注意不是“Ruby on Rails”图标）
   5. 点击“Create”(创建)
   6. 等待Cloud9完成工作空间的配置后，选择新建的工作空间并点击“Start editing”（开始编辑）


由于现在大多数Ruby程序员一般都使用两个空格作为代码缩进，所以我建议你也把文本编辑器的默认值从四个空格改成两个空格。如[图1.3](#p1.3)所示，你可以点击屏幕右上角的齿轮状的图标进入设置界面，然后选择“Code Editor(Ace)”,修改“Soft Tabs”栏的值为“2”。（这个修改是立即生效的，你不需要再去点击“Save”按钮。）

<span id="p1.2">![cloud9_new_workspace](http://i.imgur.com/H99PMkE.png)</span>
图1.2 在Cloud9上创建一个新的工作空间

<span id="p1.3">![cloud9_two_spaces](http://i.imgur.com/jvWXd1G.png)</span>  
图1.3 在Cloud9将代码缩进改成2个空格    
 
###<span id="1.2.2">1.2.2 安装Rails</span>
在[1.2.1](#1.2.1)中我们了解到,云端开发环境已经为我们准备好了所需的绝大部分软件，除了Rails。所以现在我们需要自己安装Rails。为了安装Rails,我们需要用到`gem`命令，由**RubyGems**安装包管理器提供，你需要按照[命令清单1.1](#l1.1)所示，将他们完整地输入到你的命令行终端里。（如果你使用本地开发环境，就是普通的终端窗口；如果你使用的是云端的IDE，则使用如图1.1所示的命令行终端。）  

<code><span id="l1.1" style="font-size:15px; font-weight:bold; font-family:'幼圆'">命令清单1.1： 安装指定版本的Rails软件。</span></code>   

`$ gem install rails -v 5.0`

以上命令中的`-v`符号指定了Rails的安装版本，本书的所有代码都是基于这个版本的Rails，所以务必安装该版本的Rails，否则你得到的结果可能会与本书有出入。

 
##<span id="1.3">1.3 第一个应用程序</span>  
如同计算机编程界的一贯的[传统](http://www.catb.org/jargon/html/H/hello-world.html)，我们的第一个应用程序当然是“Hello, World”程序了。在这里，我们要创建一个简单的应用程序，让它同时在我们的云IDE上（[1.3.4](#1.3.4)）和发布的网站上显示字符串“Hello， world！”。

实际上每个Rails应用都会从运行`rails new`命令开始。这个顺手的命令会在你指定的路径下创建一个Rails应用程序骨架。如果你要用自己的本地开发环境，而不是我们在[1.2.1](#1.2.1)推荐的Cloud9云端开发环境，你需要自己为你的Rails工程创建一个`workspace`工作空间，也就是在本地硬盘上新建一个名字为“workspace”的文件夹，然后通过[代码清单1.2](#l1.2)进入这个新建的文件夹。（[代码清单1.2](#l1.2)使用的是Unix风格的命令行命令`cd`和`mkdir`，如果你对这些命令不是很熟悉，可以参考[注1.3](#b1.3)）。  

    代码清单1.2： 为Rails工程新建工作空间“workspace”（云端用户不需要）

    $ cd                #进入主目录”home“ 
    $ mkdir workspace   #新建文件夹”workspace“
    $ cd workspace/     #进入文件夹”workspace“


###注1.3：Unix 命令行速成
    对于习惯使用Windows操作系统或者苹果OS X操作系统的用户来说（用户数量不多但增长很快），可能对于Unix命令行不是很熟悉。幸运的是，如果你使用我们推荐的云IDE，就可以直接使用上面提供的Unix(Linux)命令行——以标准的[shell命令行接口](https://en.wikipedia.org/wiki/Shell_(computing))[Bash](http://en.wikipedia.org/wiki/Bash_(Unix_shell))。

    命令行的基本思想很简单：用户通过简短的命令来实现大量的操作，例如新建文件夹(mkdir)，移动和复制文件（mv和cp）,以及通过切换目录来浏览文件系统内容（cd）。对于习惯使用图形界面（GUIs）的用户来说，命令行可能显得比较原始，别被表象所蒙蔽了：命令行可以说是开发者工具箱里最强有力的工具之一。实际上你会经常看到那些经验丰富的开发者在他们的电脑桌面上同时运行着好几个命令行终端程序窗口shell。

    Unix命令行博大精深，但是你只要掌握其中一些常见的Unix命令行就可以满足阅读本书的需求了，如[表格1.1](#t1.1)所示。如果你想深入了解Unix命令行，请参考Mark Bates写的《[Conquering the Command Line](http://conqueringthecommandline.com/)》（在线阅读是免费的，同时也可以购买电子书和演示视频）。


| 描述        | 命令   |  示例  |
| :--------   | :-----  | :----  |
| 列举内容     | ls |   $ ls -l |
| 新建文件夹  | mkdir \<dirname\>  |$ mkdir workspace|
| 变换目录    | cd \<dirname\>|$ cd workspace|  
| 回到上一级目录 | |$ cd .. |
| 回到主目录  | |$ cd ~ 或者 $ cd|
| 进入主目录下的文件夹 | |$ cd ~/workspace/| 
| 移动文件（重命名） | mv \<source\>  \<target>|$ mv README.rdoc README.md|
| 复制文件  | cp \<source\>  \<target>|$ cp README.rdoc README.md|
| 删除文件    | rm \<file\>|$ rm README.rdoc|  
| 删除空文件夹 | rmdir \<directory>|$ rmdir workspace/|
| 删除非空文件夹  | rm -rf \<directory>|$ rm -rf tmp/|
| 连接并显示文件内容 |cat \<file\> |$ cat ~/.ssh/id_rsa.pub| 
<span id="t1.1">表1.1：Unix常见命令行命令</span>  

接下来的步骤是使用[代码清单1.3](#l1.3)来创建我们的第一个应用程序，本地开发环境和云IDE的操作方法相同。注意在[代码清单1.3](#l1.3)中我们明确地把Rails的版本号**（_4.2.2_）**包含进来了，以此来保证我们使用和[代码清单1.1](#l1.1)中安装的Rails版本一致，然后就可以创建我们的文件夹目录结构了。（如果你输入了[代码清单1.3](#l1.3)后返回错误信息“Could not find 'railties'”，那就是说明你安装的Rails版本不对，请回头去检查你是否完全按照[代码清单1.1](#l1.1)的说明去操作）。

<code><span id="l1.3" style="font-size:12px; font-weight:bold; font-family:''">代码清单1.3： 运行Rails new（使用指定版本的Rails）。</span></code>

```
$ cd ~/workspace
$ rails _4.2.2_ new hello_app
      create
      create  README.rdoc
      create  Rakefile
      create  config.ru
      create  .gitignore
      create  Gemfile
      create  app
      create  app/assets/javascripts/application.js
      create  app/assets/stylesheets/application.css
      create  app/controllers/application_controller.rb
      .
      .
      .
      create  test/test_helper.rb
      create  tmp/cache
      create  tmp/cache/assets
      create  vendor/assets/javascripts
      create  vendor/assets/javascripts/.keep
      create  vendor/assets/stylesheets
      create  vendor/assets/stylesheets/.keep
         run  bundle install
      （译者注：如果停在这里不动，可以按Ctrl+c终止运行，然后打开hello_app/Gemfile，把
        "https://rubygems.org“替换为”https://ruby.taobao.org“，然后进入hello_app目录，重新运行”bundle install“）
Fetching gem metadata from https://rubygems.org/..........
Fetching additional metadata from https://rubygems.org/..
Resolving dependencies...
Using rake 10.3.2
Using i18n 0.6.11
.
.
.
Your bundle is complete!

Use `bundle show [gemname]` to see where a bundled gem is installed.
         run  bundle exec spring binstub --all
* bin/rake: spring inserted
* bin/rails: spring inserted
```

如[代码清单1.3](#l1.3)中所显示的一样，当我们运行命令**Rails new**后，在创建文件后，系统会自动运行命令**bundle install**。我们将从[章节1.3](#1.3.1)开始更详细的讨论这些细节。

留意一下刚才我们使用**rails**命令创建了多少文件和文件夹。这些标准的目录和文件（[图1.4](#p1.4)）是Rails众多优点之一；它让你从零开始瞬间创建了一个完整的最小网站应用程序。并且，因为所有的Rails应用程序都使用这种文件目录结构，所以你在阅读别人写的Rails代码时，可以很快明白那些代码和文件的作用。在[表格1.2](#t1.2)中列出了Rails默认的文件目录；我们将在本书的后续章节中学习这些文件和文件夹的作用。特别说明的是，我们将会在[5.2.1](#5.2.1)中学习app/assets文件夹，它是asset pipeline的一部分，用它来组织和部署一些资源，如层叠样式表（CSS）和JavaScript脚本文件，将会是前所未有的简单。

<span id="p1.4">![directory_structure_rails_3rd_edition](http://i.imgur.com/xUDCpay.png)</span>  
图1.4 新建的Rails app文件目录结构


| 文件/文件夹 | 作用  |
| :--------| :----  |
| app/     | app内核代码，包括模型，视图，控制器以及帮助文档 |
| app/assets  | 应用程序资源文件，例如CSS，JS，图片等 |
| bin/    | 二进制可执行文件| 
| config/ |配置文件 |
| db/  |数据库文件 |
| doc/ |应用程序相关说明文档 |
| lib/ | 代码模块库文件|
| lib/assets  | 代码库资源文件，例如CSS，JS，图片等|
| log/    | 应用程序日志文件|
| public/ | 可访问公共文件（比如通过浏览器访问），以及错误页面等|
| bin/rails  | 代码生成，打开终端会话，或者启动本地服务器的应用程序|
| test/ |测试程序| 
| tmp/     | 临时文件|
| vendor  | 第三方代码，如插件和gems等|
| vendor/assets| 第三方资源文件，如CSS，JS，图片等| 
| README.rdoc| 应用程序的简介文档|
| Rakefile |通过`rake`命令调用的实用任务 |
| Gemfile| 本app需要用到的Gem文件|
| Gemfile.lock|gems列表，用来确认app实用的gems是否都是同一个版本|
| config.ru |Rack中间件（[Rack middleware](http://rack.github.io/)）的配置文件|
| .gitignore| Git版本管理需要忽略的相关文档或文档格式|  
<span id="t1.2">表1.2 默认的Rails文件目录结构</span> 

###<span id="1.3.1">1.3.1 Bundler</span>  
在创建了新的Rails应用后，下一步就是使用**Bundler**来安装和包含app所需的gem。如同[章节1.3](#1.3)中描述的一样，Bundler是通过**rails**命令自动运行**bundle install**的，但是在这一节中我们将修改一些应用默认的gem,因此需要重新运行Bundler。其中包含使用文本编辑器打开**Gemfile**文件。（如果使用云IDE，用户只需在左边的文件导航栏上点击展开文件的箭头，在应用程序目录下找到**Gemfile**文件，然后再双击就可以了。）虽然具体的版本号和部分文档内容可能有些小不同，但是大体上应该和[图1.5](#p1.5)和[代码清单1.4](#l1.4)接近。（这个Gemfile中的代码是Ruby代码，但是现在不用担心看不懂Ruby语法；在[章节4](#4)）将会对Ruby有更深入的介绍。）如果左边的文件导航栏上没有出现和[图1.5](#p1.5)相同的样子，请单击屏幕右上角那个齿轮状的图标，然后选择“Refresh File Tree”（刷新文件目录树）。（一般情况下如果左边的文件目录导航栏有显示问题，只要按照上面的方法刷新一下就好。）  

<span id="p1.5">![Gem_file](http://i.imgur.com/vBFc51m.png)</span>  
图1.5 Gemfile在文本剪辑器打开的默认样式


<code><span id="l1.4" style="font-size:12px; font-weight:bold; font-family:''">代码清单1.4： 在`hello_app`应用文件夹中`Gemfile`的默认文本。</span></code>

```ruby
source 'https://rubygems.org'


# Bundle edge Rails instead: gem 'rails', github: 'rails/rails'
gem 'rails', '4.2.2'
# Use sqlite3 as the database for Active Record
gem 'sqlite3'
# Use SCSS for stylesheets
gem 'sass-rails', '~> 5.0'
# Use Uglifier as compressor for JavaScript assets
gem 'uglifier', '>= 1.3.0'
# Use CoffeeScript for .js.coffee assets and views
gem 'coffee-rails', '~> 4.0.0'
# See https://github.com/sstephenson/execjs#readme for more supported runtimes
# gem 'therubyracer', platforms: :ruby

# Use jquery as the JavaScript library
gem 'jquery-rails'
# Turbolinks makes following links in your web application faster. Read more:
# https://github.com/rails/turbolinks
gem 'turbolinks'
# Build JSON APIs with ease. Read more: https://github.com/rails/jbuilder
gem 'jbuilder', '~> 2.0'
# bundle exec rake doc:rails generates the API under doc/api.
gem 'sdoc', '~> 0.4.0', group: :doc

# Use ActiveModel has_secure_password
# gem 'bcrypt', '~> 3.1.7'

# Use Unicorn as the app server
# gem 'unicorn'

# Use Capistrano for deployment
# gem 'capistrano-rails', group: :development

group :development, :test do
  # Call 'debugger' anywhere in the code to stop execution and get a
  # debugger console
  gem 'byebug'

  # Access an IRB console on exceptions page and /console in development
  gem 'web-console', '~> 2.0.0.beta2'

  # Spring speeds up development by keeping your application running in the
  # background. Read more: https://github.com/rails/spring
  gem 'spring'
end
```

很多行都有带着“#”标识的注释；这是用来说明一些常用的gems，以及对Bundler语法的举例说明。到目前为止，我们只需要默认的gem就足够了。

除非你自己在`gem`命令中指定一个特定版本的gem，否则Bundle会自动安装最新的gem版本。情况就是这样，例如下面这个例子：

`gem 'sqlite3'`


还有两种常见的方法来指定gem版本范围，这样可以让用户通过Rails来控制版本的使用。第一种方法如下：

`gem 'uglifier', '>= 1.3.0'`

这个方法安装等于或高于1.3.0版本的`uglifier`gem（用来压缩asset pipeline中的文件），就是说，就算是高出很多的7.2版本，也还是会被安装。第二种方法如下：

`gem 'coffee-rails', '~> 4.0.0'`

这个方法安装了coffee-rails gem，其安装版本要高于4.0.0版本，但是不高于4.1版本。换种说法，“>=”符号常用来安装最新的gem，但是“~>4.0.0”表示仅安装其最小步长的发行版本（例如从4.0.0到4.0.1），而不是安装主发行版本（例如从4.0到4.1）。不幸的是，经验告诉我们，有时候即使是最小步长的版本号改变，也会导致一些问题的出现，所以在本书中我们为求稳妥，都会给所有的gem指定精确的版本号。你当然可以使用任何最新版本的gem，包括在Gemfile中使用“~>”符号（我一般只推荐高级用户使用），但是在这里要事先声明，这可能会导致本书中的某些程序出现异常。

将Gemfile中的[代码清单1.4](#l1.4)改为拥有更精确版本的如同[代码清单1.5](#l1.5)的gem。请注意，我们趁这个机会将gem sqlite3移到了后面，让他只在开发环境或者测试环境下使用（[章节7.1.1](#7.1.1)），这是为了避免它和Heroku所使用的数据库（PostgreSQL）发生冲突（[章节1.5](#1.5)）。

<code><span id="l1.5" style="font-size:12px; font-weight:bold; font-family:''">代码清单1.5：对所有Ruby gem指定精确版本的Gemfile文件。</span></code>

```ruby
source 'https://rubygems.org'

gem 'rails',                '4.2.2'
gem 'sass-rails',           '5.0.2'
gem 'uglifier',             '2.5.3'
gem 'coffee-rails',         '4.1.0'
gem 'jquery-rails',         '4.0.3'
gem 'turbolinks',           '2.3.0'
gem 'jbuilder',             '2.2.3'
gem 'sdoc',                 '0.4.0', group: :doc

group :development, :test do
  gem 'sqlite3',     '1.3.9'
  gem 'byebug',      '3.4.0'
  gem 'web-console', '2.0.0.beta3'
  gem 'spring',      '1.1.3'
end
```

当你把[代码清单1.5](#l1.5)中的代码都输入到app的“Gemfile”文件中后，就可以使用命令**bundle install**来安装所需的gem了：

```
$ cd hello_app/
$ bundle install
Fetching source index for https://rubygems.org/
.
.
.
```

命令**bundle install**的执行可能需要一段时间，一旦执行完成后，我们的网站应用程序就可以运行了。

###<span id="1.3.2">1.3.2 rails服务器</span>  
多亏我们在[1.3](#1.3)中运行了**rails new**命令，以及在[1.3.1](#1.3.1)中运行了**bundle install**命令，所以现在我们已经拥有了一个可以随时运行的网站应用程序，但是，怎么运行呢？值得高兴的是，Rails自带了一个命令行程序，或者叫做脚本程序，是可以在本地运行的一个Web服务器，用来协助我们开发程序。具体的命令取决于你所使用的开发环境：如果是本地的开发环境，你只需要运行**rails server**命令就行（[代码清单1.6](#l1.6)），如果是在Cloud9平台上，你需要提供一个额外的绑定的IP地址和端口号，Rails服务器可以用这个IP地址和端口号来使app对外部世界可见（[代码清单1.7](#l1.7)）。（Cloud9使用特殊的环境变量$IP和$PORT来动态指定IP地址和端口号。如果你想得到这些变量，在命令行中输入`echo $IP`或者`echo $PORT`。）如果你的系统提示缺少JavaScript运行时，你可以访问[execjs page at GitHub](https://github.com/sstephenson/execjs)来寻找解决方法。我特别推荐直接安装[Node.js](http://nodejs.org/)。

<code><span id="l1.6" style="font-size:12px; font-weight:bold; font-family:''">代码清单1.6：在本地机器上运行Rails服务器。</span></code> 

```
$ cd ~/workspace/hello_app/
$ rails server
=> Booting WEBrick
=> Rails application starting on http://localhost:3000
=> Run `rails server -h` for more startup options
=> Ctrl-C to shutdown server
```

<code><span id="l1.7" style="font-size:12px; font-weight:bold; font-family:''">代码清单1.7：在云端IDE上运行Rails服务器。</span></code> 

```
$ cd ~/workspace/hello_app/
$ rails server -b $IP -p $PORT
=> Booting WEBrick
=> Rails application starting on http://0.0.0.0:8080
=> Run `rails server -h` for more startup options
=> Ctrl-C to shutdown server
```

无论你使用了那种方案，我建议你新开一个命令终端选项卡，然后在新的选项卡里运行rails server，这样你还可以在第一个命令终端选项卡继续执行命令，如[图1.6](#p1.6)和[图1.7](#p1.7)所示。（如果你已经在第一个选项卡里运行了rails server，可以使用“Ctrl+C”来关闭他）。使用本地环境的用户，在浏览器中输入这个地址：[http://localhost:3000/](http://localhost:3000/);使用云端IDE的用户，打开"Share"分享面板，然后单击应用地址打开他，如[图1.8](#p1.8)。无论哪种方式，最终的输出应该和[图1.9](#p1.9)接近。

<span id="p1.6">![new_terminal_tab](http://i.imgur.com/zW49bYB.png)</span>  
图1.6 打开一个新的命令选项卡


<span id="p1.7">![rails_server_new_tab](http://i.imgur.com/CUFcgSq.png)</span>    
图1.7 在新的命令选项卡中运行Rails服务器


<span id="p1.8">![share_workspace](http://i.imgur.com/fUi8JAi.png)</span>    
图1.8 分享运行在云端工作空间中的本地服务器

<span id="p1.9">![riding_rails_3rd_edition](http://i.imgur.com/yTwqLhm.png)</span>  
图1.9 rails server的默认显示页面

可以单击“About your application's environment”（关于你的应用程序环境）可以查看这个app的相关信息。虽然可能实际的版本号会有所差异，但是页面显示应该和[图1.10](#p1.10)差不多。当然，我们不会一直需要这个默认页面，但是能看到他说明rails已经能正常工作了，这是件值得高兴的事情。我们将会在[章节1.3.4](#1.3.4)中将这个默认页面替换成我们自己设计的页面。

<span id="p1.10">![riding_rails_environment_3rd_edition](http://i.imgur.com/eP6pbyo.png)</span>  
图1.10 带有app开发环境信息的rails默认页面  

###<span id="1.3.3">1.3.3 模型-视图-控制器（MVC）</span>  
就算是刚开始，从专业的角度了解一下Rails的工作方式会对我们后面的学习有很大的帮助[图1.11](#p1.1)。你可能已经注意到，在标准的Rails应用中有一个叫**app**的文件夹，在“app/”文件夹下面有三个子文件夹：**models**，**views**，**controller**。这暗示了Rails遵循MVC结构模式，这种模式强行将“域逻辑”（也叫“业务逻辑”）和图形用户界面（GUI）相关联的输入和表现逻辑分割开来。在Web应用中，“域逻辑”一般是由用户，文章和产品等数据模型组成，而GUI只是在浏览器中的网页而已。  

当我们和Rails应用发生交互时，浏览器会发送一个**request**（请求），然后由web服务器收到并传递给Rails**contorller**（控制器），这个**controller**(控制器）将决定接下来做什么。在某些情况下，控制器将会立即根据视图（view）输出Web页面，视图就是由一个能够渲染成HTML文件的模板（译者注：这就是动态网页的原理），然后送回用户浏览器。在一般的动态网页中，控制器与模型做交互，模型是用Ruby对象来表示的应用程序的组成部分（例如帐户），负责与数据库打交道。在与模型交互之后，控制器渲染视图，然后以HTML的文件格式将网页发送至浏览器。

<span id="p1.11">![mvc_schematic](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/mvc_schematic.png)</span>  
图1.11 MVC架构模型框图

假如你现在觉得有关MVC的讨论有点抽象，不用担心；我们会在后面的篇幅里不断提及这节内容的M。[1.3.4](#1.3.4)将向你展现一个体验性的MVC应用，而在[2.2.2](#2.2.2)我们会通过玩具应用（toy app）讨论更多有关MVC的细节。最后，sample app将会用到MVC的所有知识点；从[3.2](#3.2)开始，我们将覆盖控制器（controller）和视图（view）这两个知识点，模型（model）将会在[6.1](#6.1)中有所描述，最后我们将在[7.1.2](#7.1.2)中看到它们仨快乐地一起工作。  

###<span id="1.3.4">1.3.4 Hello,world!</span>  
 
作为我们的第一个MVC架构的应用，我们要对它稍微做一点改动，添加一个控制器动作来渲染字符串“hello,world!”。（在[2.2.2](#2.2.2)中我们会了解到更多关于控制器动作的知识。）输出的结果将会是把[图1.9](#p1.9)的默认页面替换成我们将要输出的“hello,world!”页面，这也是我们这一节的目标。

顾名思义，控制器动作的意思就是定义在控制器内部的动作。我们将要召唤我们的“hello”动作，并将他放到app的控制器里。可能你已经猜到了，这个app控制器是目前我们唯一的一个app控制器，你可以通过运行下面这个命令来检测当前有哪些控制器在运行：  

`$ ls app/controllers/*_controller.rb`  

我们将从[2](#2)开始创建自己的控制器。[代码清单1.8](#l1.8)显示了控制器动作“hello”的定义，我们将使用**render**函数来返回字符串“hello,world!”。（现在先不用担心不懂Ruby语法，在[4](#4)我们将会有深入的介绍。）

<code><span id="l1.8" style="font-size:12px; font-weight:bold; font-family:''">代码清单1.8：向ApplicationController中添加hello动作。</span></code>  
`app/controllers/application_controller.rb`

```ruby
class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception

  def hello
    render text: "hello, world!"
  end
end
```

我们已经定义了一个动作(函数)**hello**来返回我们需要的字符串，接下来需要告诉Rails使用这个动作来代替之前如[图1.10](#p1.10)的默认页面。为了达到这个目的，我们需要修改Rails的路由（router），它位于控制器之前，[图1.11](#p1.11)，由它来决定浏览器发送来的请求发送给那个控制器。（为了简洁，在[图1.11](#p1.11)中我没有画出路由，但是别担心，我们会在[2.2.2](#2.2.2)中做更多的讲解。）在这里，我们要修改的是默认的首页，也就是根路由，这个根路由将决定根URL显示哪个页面。因为这个URL是形如**http://www.example.com/**样的网址（最后一个"/"后没有内容），所以根URL一般都会简化为用“/”来表示。

如同[代码清单1.9](#l1.9)所示，Rails的路由文件中（config/routes.rb）包含了一行怎么表示根路由的注释。这里，“welcome”是控制器的名字，而“index”是控制器中动作的名称。删除注释符“#”，然后将代码改成如同[代码清单1.10](#l1.10)所示的一样，这样Rails就知道根路由是指向Applicationcontroller的**hello**动作了。（如同[1.1.2](#1.1.2)中提到的，代码清单中的垂直省略号是用来省略某些代码的，不能整段复制过去直接运行。）

<code><span id="l1.9" style="font-size:12px; font-weight:bold; font-family:''">代码清单1.9：默认的根路由文件（注释掉的）。</span></code>  
`config/routes.rb`  

```ruby
Rails.application.routes.draw do
  .
  .
  .
  # You can have the root of your site routed with "root"
  # root 'welcome#index'
  .
  .
  .
end
```

<code><span id="l1.10" style="font-size:12px; font-weight:bold; font-family:''">代码清单1.10：设置根路由。</span></code>  
`config/routes.rb`
```ruby
Rails.application.routes.draw do
  .
  .
  .
  # You can have the root of your site routed with "root"
  root 'application#hello'
  .
  .
  .
end
```

使用了[代码清单1.8](#l1.8)和[代码清单1.10](#l1.10)之后，根路由就会返回有字符串“hello,world!”的页面，就像我们期待的那样（[图1.12](#p1.12)）

<span id="p1.12">![HelloWorld](http://i.imgur.com/ohQ9Zfn.png)</span> 
图1.12 在浏览器中显示“hello,world!”


##<span id="1.4">1.4 使用Git来进行版本管理</span> 
既然我们现在已经有了一个可以正常运行的Rails应用，接下来我们将花点时间来对我们的源代码进行版本控制。从技术上来讲这步是可选的，但是很多经验丰富的软件开发者都认为这是非常必要的。版本管理系统可以使我们很方便的追踪项目中哪些地方的代码修改过了，同时也使得大家的合作变得简单，还可以回滚一些无意的错误（例如不小心删掉了某些重要文件）。了解如何使用版本管理系统是对每个专业级的软件开发者的基本要求。

虽然有很多版本控制软件，但是Rails社区都倾向于使用[Git](http://git-scm.com/),最初是由Linus Torvalds（Linux操作系统内核的作者）为了保存Linux内核而开发的分布式版本控制系统。Git本身就是一个大的主题，在这里我们只肤浅的介绍一下，但是在网上有很多很好的免费学习资源；这里我特别推荐[Bitbucket 101](https://confluence.atlassian.com/display/BITBUCKET/Clone+your+Git+repository+and+add+source+files)作为入门学习，然后深入的学习下Scott Chacon写的《[Pro Git](http://git-scm.com/book)》。我之所以强烈推荐你将代码源程序放在Git的版本管理上，不只是因为在Rails领域大家都这么干，还因为那样会让你可以非常方便地备份和分享你的代码（[章节1.4.3](#1.4.3)），同时还可以将你在本书学到的第一个Rails应用程序部署到服务器上（[章节1.5](#1.5)）。

###<span id="1.4.1">1.4.1 安装和设置Git</span>

[章节1.2.1](#1.2.1)中推荐的云IDE已经默认安装了Git，所以不需要再次安装了。然而，如果你使用的是本地的开发环境，则你可以在网站[InstallRails.com](http://installrails.com/)（[1.2](#1.2)）上找到对应的操作系统安装Git的方法。

**首次运行时所需的系统设置**
 
在使用Git之前，你需要做一些系统设置，对于每台电脑，这些设置只需做一次。

```
$ git config --global user.name "你的名字"
$ git config --global user.email 你的email地址
$ git config --global push.default matching
$ git config --global alias.co checkout
```

注意，在以上的设置中你所使用的名字和email地址在你自己公开的Git仓库上是对外公开的。（以上的命令其实只有前面两行是必须的。第三行命令是为了保证向前兼容未来发布的Git版本。第四行命令是可选的，就是你可以使用**co**来代替更冗长的**checkout**命令。为了最大限度的兼容那些没有设置过**co**的系统，在本书中我们依然使用**checkout**命令，但是实际开发中我都是用**git co**。）

**首次使用仓库所需的设置**

现在我们要做的是每次新建仓库（有时候会把仓库（repository）简称`repo`）都要执行的一步。进入第一个应用程序的的根目录下，初始化我们的新仓库。

```
$ git init
Initialized empty Git repository in /home/ubuntu/workspace/hello_app/.git/
```

下一步将所有的项目文件添加到仓库里，使用命令`git add -A`:

`$ git add -A`

这个命令会将当前文件夹下除了那些文件名匹配一定模式的文件以外的其他所有文件添加到Git仓库中，需要忽略的文件在**.gitignore**中指定，可以使用通配符。**rails new**命令会自动生成一个适合Rails项目的**.gitignore**文件，但是你也可以添加其他你想要忽略的文件匹配模式。

这些刚加入仓库的文件会先放在一个叫暂存区的地方，这个地方用来存放项目中那些修改过，等待提交的文件。你可以使用**status**命令来查看暂存区里都有哪些文件：

```
$ git status
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

  new file:   .gitignore
  new file:   Gemfile
  new file:   Gemfile.lock
  new file:   README.rdoc
  new file:   Rakefile
  .
  .
  .
```

（这个输出列表会很长，所以我用垂直的点代表省略的输出。）  

如果你想告诉Git你要保留这些更改，你需要使用**commit**命令：

```
$ git commit -m "Initialize repository"
[master (root-commit) df0a62f] Initialize repository
.
.
.
```

`-m`标志让你加上自己需要的注释；如果你省略了`-m`标志，那么Git会启动系统的默认文本编辑器让你输入注释内容。（本书中的所有例子都会使用`-m`标志）。

请注意，使用Git commit命令所提交的修改内容是仅仅保存在本地的机器上的。在[章节1.4.4](#1.4.4)中我们将会看见怎样将修改的内容放到远端的仓库（使用**git push**命令）。

还有，你可以使用**log**命令来查看你所提交的内容日志：

```
$ git log
commit df0a62f3f091e53ffa799309b3e32c27b0b38eb4
Author: Michael Hartl <michael@michaelhartl.com>
Date:   Wed August 20 19:44:43 2014 +0000

    Initialize repository
```

如果你的仓库提交历史很多，有可能你需要使用到**q**命令来退出。

###<span id="1.4.2">1.4.2 使用Git给你带来什么好处？</span>

如果你之前从来没有用过版本管理工具，那你可能现在还不能完全体会这个工具会给你带来什么好处。让我来给你举一个例子：我们假定你不小心犯了些错误，比如删除了整个核心文件夹`app/controllers`。

```
$ ls app/controllers/
application_controller.rb  concerns/
$ rm -rf app/controllers/
$ ls app/controllers/
ls: app/controllers/: No such file or directory
```

为了进行这个“一不小心”的失误，我们要用到一些Unix命令，首先，使用`ls`命令来显示`app/controllers/`文件夹所包含的内容，然后再使用`rm`命令将其删除（[表格1.2](#t1.2)）。`-rf`标记的意思是“强制递归”，也就是说要递归删除所有的文件，文件夹，子文件夹等等，一直删除到没有任何内容为止，而且在删除过程中不会有任何提示。

让我们查看一下状态，看看发生了什么：

```
$ git status
On branch master
Changed but not updated:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

      deleted:    app/controllers/application_controller.rb

no changes added to commit (use "git add" and/or "git commit -a")
```

在此我们可以看到有一个文件被删除了（application_controller.rb），但是目前为止这些改变还只发生在“工作树”中，还没有提交到仓库中。这意味着我们可以使用`checkout`命令和`-f`标记来强行撤销当前修改：

```
$ git checkout -f
$ git status
# On branch master
nothing to commit (working directory clean)
$ ls app/controllers/
application_controller.rb  concerns/
```

被误删的文件和文件夹都恢复了。是不是松了一口气？

###<span id="1.4.3">1.4.3 Bitbucket</span>

既然我们已经把工程文件纳入了Git的版本管理中，现在是时候把他推送到远程端[Bitbucket](http://www.bitbucket.com/)上了，[Bitbucket](http://www.bitbucket.com/)是一个可以用来托管和分享Git仓库的网站。（这本书之前的版本使用的仓库管理网站是[GitHub](http://www.github.com/)，你可以从[注1.4](#b1.4)中了解到我们更换的原因。）将你的Git仓库副本保存到Bitbucket服务器上有两个目的：这是一个完整的代码备份（包括你的历史提交数据），同时也为将来你与其他人共同开发维护这个项目提供了很多便利。

    ####<span id="t1.4">注1.4 Bitbucket</span>

    目前世界上最受欢迎的两个Git仓库托管网站就是GitHub和Bitbucket。这两个网站有很多相似之处：两个网站都可以托管仓库和协作开发，同时也提供非常便利的浏览和搜索代码仓库的功能。对于本书来说，他们两最重要的区别在于：GitHub为开源的项目提供无限制的仓库数量，但是对私有项目仓库收费，而Bitbucket允许无限的免费的私有项目仓库，仅当项目的协作人数（项目组员）超过一定数量时才收费。至于你自己需要哪种类型的代码托管服务，要取决于你自己的实际项目需求。

    本书的前几个版本使用GitHub做代码托管仓库，是因为他有很多好用的开源项目工具，但是随着我对安全和隐私的注重，我认为还是推荐将所有的web应用仓库设为私有比较好。原因是web应用仓库里存有一些潜在的敏感信息，比如用户密钥和密码，这可能会给那些使用这些代码来运行的网站的网站带来安全隐患。当然，我们有一些办法来处理这些安全问题（例如使用Git的忽略功能），但是这很容易遗漏或犯错，同时也需要比较高的专业素养。

    本书中生成的源代码可以公开到网上，但是对于其他实际的工程来说就太危险了。所以，为了保证安全性，我们默认使用私有的仓库托管网站。GitHub上私有项目收费而Bitbucket提供免费的私有项目仓库，显而易见，对于我们的需求来说Bitbucket比GitHub要合适一些。


Bitbucket很容易上手：

1. 在Bitbucket上[注册](https://bitbucket.org/account/signup/)一个账号，如果已经有就跳过此步骤；  
2. 将公钥复制到剪贴板。使用云端IDE的用户可以通过[代码清单1.11](#l1.11)浏览公钥，也就是使用**cat**命令，然后可以直接对公钥进行选择和复制。如果你使用本地系统开发环境，当按照[代码清单1.11](#l1.11)执行命令后却没有任何输出，我推荐你查看一下这篇文章的说明：[怎样添加公钥](https://www.railstutorial.org/book/beginning#uid72)。
3. 将你的公钥添加到Bitbucket上：通过点击屏幕右上角的头像，选择“manage account”（管理账号），然后选择“SSH keys”（SSH 密钥）（[图1.13](#p1.13)）。

<code><span id="l1.11" style="font-size:12px; font-weight:bold; font-family:''">代码清单1.11：使用`cat`命令打印公钥。</span></code>

```
$ cat ~/.ssh/id_rsa.pub

```


<span id="p1.13">![addpublickey](http://i.imgur.com/omTRVA6.png)</span>  
图1.13 添加SSH公钥

一旦你添加好公钥，就可以点击“Create”（创建），创建一个新的仓库，如[图1.14](p1.15)所示。当填写项目信息时，一定要勾选“This is a private repository”（这是一个私有仓库）。点击“Create repository”（创建仓库）后，按照“Command line> I have an existing project”（命令行>我已经有一个仓库）下面的指示操作，如[代码清单1.12](#l1.12)所示。（如果显示结果和[代码清单1.12](#l1.12)不一样，那有可能是你的公钥没有设置正确，我建议你重新尝试一遍）。当推送仓库时，如果你看到问题“Are you sure you want to continue connecting (yes/no)?”（你确定需要继续连接吗？（是/否）），选择yes。

<span id="p1.14">![create_first_repository_bitbucket](http://i.imgur.com/4joaDdi.png)</span>   
图1.14 为第一个app创建Bitbucket仓库


<code><span id="l1.12" style="font-size:12px; font-weight:bold; font-family:''">代码清单1.12：添加Bitbucket仓库并将本地内容推送到上面。</span></code>

```
$ git remote add origin git@bitbucket.org:<username>/hello_app.git
$ git push -u origin --all # 第一次把代码推送到远程仓库

```

[代码清单1.12](#l1.12)中的命令首先告诉Git你要把Bitbucket添加为仓库源，然后将你的仓库推送到远端的源地址去。（不用担心标记`-u`的具体作用，如果你对此感兴趣，可以搜索一下“git set upstream”（git上载））。当然，你需要把\<username\>替换成你自己的用户名。例如，我自己是这么写的： 

```
$ git remote add origin git@bitbucket.org:mhartl/hello_app.git

```

运行的结果是Bitbucket上的hello_app代码仓库，包含了文件浏览，历史提交注释，以及其他很多很好的东西（[图1.15](#p1.15)）。


<span id="p1.15">![bitbucket_repository_page](http://i.imgur.com/iIRk6SR.png)</span>  
图1.15 Bitbucket仓库页面


###<span id="1.4.4">1.4.4 分支，编辑，提交，合并</span>

如果你按照[1.4.3](#1.4.3)的步骤操作，你可能已经注意到了Bitbucket没有从仓库中自动检测到README.rdoc文件，而是在仓库主页上提示你缺少了README文档（[图1.16](#p1.16)）。这说明rdoc后缀的文件格式不是Bitbucket通常支持的格式，确实，我和大部分我认识的开发者都更倾向于使用Markdown格式的文档。在这一节中，我们要将README.rdoc改成README.md，顺便利用这个机会我们会往这个README文档里添加一些针对本书的特别说明。在这个过程中，我们将第一次看见我推荐使用的Git的工作流程，包括创建分支、编辑、提交、合并等动作。


<span id="p1.16">![bitbucket_no_readme](http://i.imgur.com/NzajpHm.png)</span>  
图1.6 Bitbucket提示缺少README文档

####分支

Git的分支功能强到令人难以相信的程度，它可以以非常高效的方式将整个仓库复制下来，修改副本不会对父仓库（master分支）有任何影响。在大多数情况下，父仓库就是*主*分支，然后我们可以使用**checkout**命令加上**-b**标记来创建一个新的分支：

```
$ git checkout -b modify-README
Switched to a new branch 'modify-README'
$ git branch
  master
* modify-README
```

第二个命令**git branch**可以列出本地仓库的所有分支，星号”*“表示我们当前所在的分支。请注意**git checkout -b modify-README**命令既创建了一个新的分支又把它设为当前分支，正如在**modify-README**分支前加了星号所暗示的一样。（如果你设置了[1.4](#1.4)的`co`缩写命令，你可以使用这个命令代替刚才的命令：`git co -b modify-README`）。

当一个项目由多个人协作开发时，分支的全部价值才真正体现出来，但是就算只有一个开发者，就像本书的例子一样，分支也是非常实用的。尤其，我们要对把主题分支的修改和主分支
隔离开来，这样就算我们真的把程序搞崩溃了，我们还可以通过取出（checkout）主分支，放弃我们所做的修改，然后将其他主题分支删掉。在这节的最后我们将会具体说明。

另外，一般像这种小改动是没有必要新建一个分支的，但是这对于新手来说是一个非常好的机会去养成这种好习惯。

####编辑

当新建了一个主题分支后，我们要编辑一下，让它的信息更加丰富。相比默认的RDoc文件，我更喜欢用[Markdown markup语言](http://daringfireball.net/projects/markdown/)来做这件事，而且如果你使用`.md`后缀的文件格式，Bitbucket能够很好地自动将其格式化。所以，首先我们要使用Git版本提供的Unix`mv`（移动）命令来修改文件后缀名：

```
$ git mv README.rdoc README.md

```

然后将[代码清单1.113](#l1.13)的内容填写到README.md文档中。

<code><span id="l1.13" style="font-size:12px; font-weight:bold; font-family:''">代码清单1.13：新的README文档：README.md。</span></code>

```
# Ruby on Rails Tutorial: "hello, world!"

This is the first application for the
[*Ruby on Rails Tutorial*](http://www.railstutorial.org/)
by [Michael Hartl](http://www.michaelhartl.com/).
```

####提交

当修改好EADME文档以后，我们可以查看一下我们的分支：

```
$ git status
On branch modify-README
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

  renamed:    README.rdoc -> README.md

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

  modified:   README.md
```

在这个情况下，我们本来可以用[1.4.1.2](#1.4.1.2)中的`git add -A`命令，但是`git commit`命令提供了`-a`标记，可以用来将所有现有的修改过的文件全部提交（包括由`git mv`命令生成的文件，Git不认为那是新的文件）：

```
$ git commit -a -m "Improve the README file"
2 files changed, 5 insertions(+), 243 deletions(-)
delete mode 100644 README.rdoc
create mode 100644 README.md
```

小心错误地使用`-a`标记，如果在提交之前你新生成了一些文件，这时你还是需要先用`git add -A`告诉Git有新的文件需要添加。

注意，我们所写的提交信息是用的现在时（严格来说应该是[祈使句](https://en.wikipedia.org/wiki/Imperative_mood)）。Git将我们一系列的提交模拟为补丁，在这种情况下说明提交现在做什么，比他过去做了什么更合理。而且，这种用法和Git命令产生的提交信息相匹配。更多信息请参考”[亮瞎眼的新提交样式](https://github.com/blog/926-shiny-new-commit-styles)“。

####合并

当我们修改好所需文档后，我们需要将其合并到主分支上：

```
$ git checkout master
Switched to branch 'master'
$ git merge modify-README
Updating 34f06b7..2c92bef
Fast forward
README.rdoc     |  243 --------------------------------------------------
README.md       |    5 +
2 files changed, 5 insertions(+), 243 deletions(-)
delete mode 100644 README.rdoc
create mode 100644 README.md
```

请注意，Git的输出有时候会包含一些编号，类似”34f06b7“，这是Git内部用来区分不同的仓库的代号。你得到的输出可能和上面的不尽相同，但是大体上应该一样。

当你合并这个修改之后，你就可以使用`git branch -d`命令来删除这个话题分支了：

```
$ git branch -d modify-README
Deleted branch modify-README (was 2c92bef).
```

这一步是可选的，其实完整的保留主题分支是非常平常的。这样你就可以在主题分支和主分支之间来回切换，并在合适的时候将修改合并到主分支上。

如同上面所说的一样，你也可以放弃对这个话题分支的修改，这个时候要用到`git branch -D`命令：

```
# 仅作为演示使用；如果你没把这个分支弄砸，没必要把他灭了
$ git checkout -b topic-branch
$ <把代码修改的一团糟>
$ git add -A
$ git commit -a -m "Major screw up"
$ git checkout master
$ git branch -D topic-branch
```

不像`-d`标记，这里的`-D`标记将删除话题分支，就算是修改的内容没有合并到主分支，还是会被无情地灭掉。

####推送

我们现在已经更新了README文档，接下来我们可以将这些修改推送到Bitbucket远程仓库，去看看结果。在[1.4.3](#1.4.3)中我们已经推送过一次了，而大部分的系统都会有记忆功能，因此我们可以省略`origin master`，直接运行`git push`就可以了：

```
$ git push

```

就像我们在[1.4.4.2](1.4.4.2)中所说的一样，Bitbucket会自动解析Markdown文档（[图1.17](#p1.17)）。


<span id="p1.17">![new_readme_bitbucket](http://i.imgur.com/4a2xlhU.png)</span>  
图1.17 README文档修改为Markdown格式


##<span id="1.5">1.5 部署</span>

虽然我们才开始第一章，但是现在我们已经要准备把我们的Rails应用（几乎是空的应用）部署到生产环境中去。这一步是可选的，但是越早部署程序，我们就能越早发现开发过程中存在的问题。还有一种方法是在开发环境花了很大的力气把程序做好，然后再部署，这种方法往往在准备发布时会面临很多令人头疼的问题。

过去部署Rails应用程序很麻烦，但是在过去的几年时间里，Rails开发生态系统已经日臻成熟，现在已经有很多非常棒的工具可选。包括通过运行[Phusion Passenger](http://www.modrails.com/)（Apache和Nginx web服务器的一个模块）可以实现主机共享或虚拟私有服务器VPS， [Engine Yard](http://engineyard.com/)和[Rails Machine(http://railsmachine.com/)]公司提供的一站式部署服务，以及云部署服务提供商[Engine Yard Cloud](http://cloud.engineyard.com/)，[Ninefold](https://ninefold.com/)和[Heroku](http://heroku.com/)等等。

我最喜欢用Heroku来部署Rails应用，它主要是针对Rails和其他web应用的部署平台。Heroku让Rails部署变得异常简单——只需要你的源代码是用Git版本管理就行。（这是为什么要按照[1.4](#1.4)的步骤来配置Git的另外一个原因，如果你还没做，那赶紧吧。）另外，还有其他一些目的，包括对本书而言，Heroku的免费空间超乎想象。确实，本书的前两个版本都得益于Heroku提供的免费空间，Heroku从来没有因为本书的读者申请的那数百万个服务器空间向问我要一分钱。

本节的后续部分将专注于把Rails应用部署到Heroku上。其中某些操作方法相当高级，所以如果有些细节理解不了，不用担心；重要的是我们最终会把我们的第一个Rails应用部署到线上。

###<span id="1.5.1">1.5.1 Heroku 设置</span>

Heroku使用的是[PostgreSQL](http://www.postgresql.org/)数据库（全称是“post gres cue ell”），这意味着我们要把**pg**gem添加到生产环境中，好让Rails可以与Postgres通信：

```
group :production do
  gem 'pg',             '0.17.1'
  gem 'rails_12factor', '0.0.2'
end
```

注意还需要**rails_12factor**gem，这是Heroku用来服务类似图片和CSS文件等静态资源的。最终**Gemfile**文档应该如[代码清单1.14](#l1.14)所示。

<code><span id="l1.14" style="font-size:12px; font-weight:bold; font-family:''">代码清单1.14：添加了必须的Gem以后的Gemfile文档。</span></code>  

```ruby
source 'https://rubygems.org'

gem 'rails',        '4.2.2'
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

为了部署生产环境，我们需要运行**bundle install**来安装Rails应用程序所需的Gem，**--without production**表示不在本地服务器安装属于production组的Gem，（这里指的是*pg*和*rails_12factor*）：

```
$ bundle install --without production

```

因为在[代码清单1.14](#l1.14)中添加的Gem只用于生产环境，现在执行这条命令不会再本地安装任何额外的Gem，但是我们需要把`pg`和`rails_12factor`添加到`Gemfile.lock`文档中。我们可以使用下面的方法来提交这个修改：

```
$ git commit -a -m "Update Gemfile.lock for Heroku"

```

接下来我们需要创建和配置一个新的Heroku账号。当然第一步是[注册Heroku账号](http://api.heroku.com/signup)。然后看看你的系统里是否已经安装了Heroku的命令行客户端：

```
$ heroku version

```

使用云IDE的读者应该可以看到Heroku的版本号，这表示Heroku的命令行已经可以使用了。如果是本地的开发环境，你可能需要使用[Heroku Toolbelt](https://toolbelt.heroku.com/)来安装。

一旦你确认Heroku命令行工具已经安装完毕，接下来就可以使用heroku命令来登录并添加你的SSH密钥：

```
$ heroku login
$ heroku keys:add
```

最后，使用`heroku create`命令会将我们的程序部署在Heroku服务器（[代码清单1.15](l1.15)）。

<code><span id="l1.15" style="font-size:12px; font-weight:bold; font-family:''">代码清单1.15：在Heroku上新建一个app。</span></code> 

```
$ heroku create
Creating damp-fortress-5769... done, stack is cedar
http://damp-fortress-5769.herokuapp.com/ | git@heroku.com:damp-fortress-5769.git
Git remote heroku added
```

heroku命令为我们的应用创建一个新的二级域名，可以直接浏览。目前那里还没任何内容，让我们赶紧部署吧。

###<span id="1.5.2">1.5.2 Heroku 部署，第一步</span>  

要部署这个应用，第一步是使用Git将主分支（master）推送到Heroku：

```
$ git push heroku master

```

（你可能会看到一些警告信息，你可以暂时忽略他们。我们会在[7.5](#7.5)深入探讨这些问题）。

###<span id="1.5.3">1.5.3 Heroku 部署，第二步</span> 

没有第二步！我们已经基本完成了。为了看看你刚部署的应用，访问你刚才运行`heroku create`（例如[代码清单1.15](#l1.15)）时输出的那个网址。（如果你使用的是本地的开发环境，你可以使用命令**heroku open**。）结果如[图1.18](#p1.18)所示。当然这个页面看起来和[图1.12](#p1.12)其实是一样的，但是这可是运行在生产服务器上的在线网页哦。

<span id="p1.18">![heroku_app_hello_world](http://i.imgur.com/bKz86sl.png)</span>  
图1.18 第一个运行在Heroku上的Rails应用


###<span id="1.5.4">1.5.4 Heroku 命令</span>  

Heroku有很多[命令](http://devcenter.heroku.com/heroku-command)，在本书我们仅仅介绍一些简单的命令。接下来让我们花一分钟的时间来学习其中的一个命令，它的作用是重命名应用，如下所示：

```
$ heroku rename rails-tutorial-hello

```

千万不要再用这个“rails-tutorial-hello”的名字了，因为我已经占用它了！实际上，你现在还不需要为这花太多心思，直接使用默认的名字就行了。如果你实在想重命名你的应用，你可以使用一些随机生成的，或者意思比较模糊的二级域名，例如下面的：

> hwpcbmze.herokuapp.com  
> seyjhflo.herokuapp.com  
> jhyicevg.herokuapp.com

使用这种随机生成的二级域名的好处是，别人要想访问你的网站，只有你把地址给他了他才可以找到。顺便提一下，为了让你了解到Ruby的简洁强大，下面的代码是我用来生成随机二级域名的：

```ruby
('a'..'z').to_a.shuffle[0..7].join

```

很棒对吧？）

除了二级域名意外，Heroku还支持客户自己的域名。（实际上，本书的[网站](http://www.railstutorial.org/)就是部署在Heroku上的；如果你是在线阅读本书，你现在就在浏览一个托管在Heroku服务器上的网站！）你可以通过[Heroku相关文档](http://devcenter.heroku.com/)来获取客户自定义域名与其他相关话题文章。

##<span id="1.6">1.6 总结</span> 

在这一章我们经历了很多知识点：开发环境的安装和设置，版本控制，以及部署。在下一章，我们将在第一章的基础上建立一个拥有数据库的玩具应用，这将让我们了解Rails到底能做些什么事情。

如果你愿意可以将你自己的进度分享到推特或者脸书上，格式如下：

[I’m learning Ruby on Rails with the @railstutorial! http://www.railstutorial.org/](http://www.railstutorial.org/)

而且我推荐你加入[Rails教程邮件列表](http://www.railstutorial.org/#email)，当有新的更新会及时通知你（还有优惠码）。

###<span id="1.6.1">1.6.1 这一章我们学到了什么</span> 

  * Ruby on Rails是基于Ruby语言的一款Web开发框架；  
  * 使用云IDE进行Rails安装，生成网站应用，以及编辑文档等；  
  * Rails提供了命令行工具`rails`，可以用来新建应用（rails new）以及运行本地服务器（rails server）。  
  * 我们添加了一个控制器动作，同时修改了根路由，以此来创建了一个“hello,world!”的应用；  
  * 我们使用Git进行版本管理，这可以有效地保护代码数据。同时还把代码推送到了Bitbucket的私有仓库上；  
  * 我们将应用部署到了Heroku的生产服务器上。


##<span id="1.7">1.7 练习题</span>  

注意：练习题答案手册包含了本书的所有练习题详细解答，当购买本书时会免费赠送。详情请查看官网www.railstutorial.org。

 1. 修改[命令清单1.8](l1.8)中的hello动作，用“hola,mundo!”替换“hello,world!”。附加任务：使用倒置的感叹号，例如“?Hola, mundo!”（[图1.19](#p1.19)），来证明Rails同时也支持非ASCII码字符。
 2.在hello动作下面添加一个新的动作，名字是“goodbye”，然后把这个字符串渲染到网页上：“goodbye,world!”。然后修改路由文档（[代码清单1.10](#p1.10)），这样根路由就会改成指向goodbye动作（[图1.20](#p1.20)）。

<span id="p1.19">![mundo](http://i.imgur.com/2ygvDgQ.png)</span>  
图1.19 将根路由的返回改成“?Hola, mundo!”


<span id="p1.20">![goodbyeworld](http://i.imgur.com/4qWanZq.png)</span>  
图1.20 修改根路由返回“goodbye,world!”
















































































