# 第五章 填充布局

在第四章短暂的Ruby旅行中，我们学到了包含网页的样式表（CSS文件）进入示例应用程序（4.1节），但是（如4.3.4节里说明的）样式表文件里还不包含
任何CSS。在这章里我们将开始通过纳入一个CSS框架进入我们的应用程序，然后我们添加一些自定义格式。我们也将开始用到目前我们创建的链接填写布局（例如主页和关于页面），（5.1节）。沿着这条路，我们将学习部件（partial），Rails路由，和资源管线（asset pipeline），包括Sass的介绍（5.2节）我们将通过让用户在我们网站注册（5.4节），向前迈出重要的第一步。

在这章大部分的变化是在示例应用程序的网站布局文件添加和编辑文档，（依据注3.3里的指导原则）是确定一般不需要测试驱动的，或者甚至根本不需要测试。因此，我们将花费大部分时间在我们的文本编辑器和浏览器里，使用TDD仅仅是为了添加一个联系页面（5.3.1节）。不过，我们将
添加一个重要的新测试，编写我们第一个集成测试来检查最后的布局文件是正确的（5.3.4节）。

## 5.1 添加一些结构
本书是一本网页开发的书，不是网页设计，但是工作在一个看上去非常难看的应用程序上将是很令人沮丧的，所以在这节我们会给布局文件添加一些样式。另外，使用自定义CSS规则，我们将使用Bootstrap，这个Twitter公司开发的一个开源的网页设计框架。我们也会给我们的代码增加一些样式，可以说，一旦布局文件变得凌乱不堪，就使用partial来整理它。

当构建网页应用程序时，尽可能早得对用户接口有总体的规划常常很有用，相关内容遍及本书剩余的部分。因此我将使用mockup（在网页环境，常常叫框架图），它就是最终应用将看起来什么样的。在这章，我们将主要开发3.2节介绍的静态网页，包括网站的标志、导航栏、网站的底部。
这些页面里最重要的框架图，即主页，如图5.1。你能看见最终的结果如图5.7。你将注意到在细节上有些不同--例如，我们将通过在网页上添加一个Rails标志结束--没有关系，因为框架图不需要一模一样。

![图5.1：示例程序主页的框架图](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/home_page_mockup_3rd_edition.png)

和平常一样，假如你正使用Git作为版本控制，现在正是创建新分支的好时候：
```
$ git checkout master
$ git checkout -b filling-in-layout
```

### 5.1.1 网站导航
作为朝示例程序添加链接和样式的第一步，我们将更新网站布局文件**application.html.erb**（上一次看见是在代码清单4.3中）用另外的HTML结构。这包含一些另外的分割，一些CSS类，和我们网站导航的开始。在代码清单5.1里的是所有的文件内容；后面紧跟着就解释各种各样的片段。假如你迫不及待，你可以看看图5.2里的效果。（说明：仍然不是很满意。）

```ruby
代码清单 5.1： 添加了结构的网站布局文件。
# app/views/layouts/application.html.erb
<!DOCTYPE html>
<html>
  <head>
    <title><%= full_title(yield(:title)) %></title>
    <%= stylesheet_link_tag 'application', media: 'all',
                                           'data-turbolinks-track' => true %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
    <%= csrf_meta_tags %>
    <!--[if lt IE 9]>
      <script src="//cdnjs.cloudflare.com/ajax/libs/html5shiv/r29/html5.min.js">
      </script>
    <![endif]-->
  </head>
  <body>
    <header class="navbar navbar-fixed-top navbar-inverse">
      <div class="container">
        <%= link_to "sample app", '#', id: "logo" %>
        <nav>
          <ul class="nav navbar-nav navbar-right">
            <li><%= link_to "Home",   '#' %></li>
            <li><%= link_to "Help",   '#' %></li>
            <li><%= link_to "Log in", '#' %></li>
          </ul>
        </nav>
      </div>
    </header>
    <div class="container">
      <%= yield %>
    </div>
  </body>
</html>
```
让我们仔细看看代码清单5.1里面的新内容，如在3.4.1节简单提过的，Rails默认使用HTML5（如文档类型
**<!DOCTYPE html>**)；因为HTML5标准相对较新，一些浏览器（尤其旧版的Internet
Explore)不能很好的支持他，所以我们包含进一些Javascript代码（著名的“HTML5 shim（或者shiv）”），解决这个问题:

```ruby
<!--[if lt IE 9]>
  <script src="//cdnjs.cloudflare.com/ajax/libs/html5shiv/r29/html5.min.js">
  </script>
<![endif]-->
```
有点奇怪的语法
```ruby
<!--[if lt IE 9]>
```
假如微软IE浏览器版本低于9（**if lt IE 9**）将被包含进来。奇怪的**[if lt IE 9]**语法不是Rails的部分；它实际上是IE浏览器支持的条件注释，仅仅是为了这种情形。它也是个好东西，因为它意味着我们可以仅仅为版本低于9的IE浏览器才引入这个文件，也不会影响其他浏览器，例如Firefox，Chrome，和Safari。

下一节为网站的（纯文本）标志，包含网页头部、几个段落（使用**div**标签）和几个导航链接：
```ruby
<header class="navbar navbar-fixed-top navbar-inverse">
  <div class="container">
    <%= link_to "sample app", '#', id: "logo" %>
    <nav>
      <ul class="nav navbar-nav navbar-right">
        <li><%= link_to "Home",   '#' %></li>
        <li><%= link_to "Help",   '#' %></li>
        <li><%= link_to "Log in", '#' %></li>
      </ul>
    </nav>
  </div>
</header>
```
这里**header**用来表明网页顶部的内容。我们给**header**标签定义三个CSS类，叫**navbar，navbar-fixed-top和navbar-inverse**，用空格分开：
```html
<header class="navbar navbar-fixed-top navbar-inverse">
```
所有的HTML元素都可以分配class和id；这些只是标记，对CSS（5.1。2节）样式化有用。它们主要的不同是class可以在页面重复使用，而id仅可以用一次。在当前的例子中，导航栏的class对Bootstrap框架有特殊的意义，我们将在5.1.2节安装和使用它。

在**header**标签里，我们看看**div**的标签：
```html
<div class="container">
```
**div**标签是HTML常用的分隔标记。除了把文档分成不同的部分，它不会再做任何其他的事情。在老式的HTML里，**div**标签几乎是网页唯一的分隔标记，HTML5增加了**header，nav，和section**，许多应用程序用它们来分隔。在这个例子中，**div**也有一个CSS
类（**container**）。和**header**标签的类一样，这个类对Bootstrap来说有特殊的意义。

看过div以后，我们与一些内嵌的Ruby邂逅了：
```erb
<%= link_to "sample app", '#', id: "logo" %>
<nav>
  <ul class="nav navbar-nav navbar-right">
    <li><%= link_to "Home",   '#' %></li>
    <li><%= link_to "Help",   '#' %></li>
    <li><%= link_to "Log in", '#' %></li>
  </ul>
</nav>
```

这段代码使用了Rails的辅助方法**link_to**来创建链接（在3.2.2节里我们用锚点直接创建的）；**link_to**的第一个参数是链接文本，第二个是URL。我们会在5.3.3节里具名路由来填充URL，但是现在我们使用在网页设计里经常使用的锚点URL'#'。第三个参数是可选的哈希，在这里把id等于**logo**的CSS内容添加到应用程序的链接中去。（其他三个链接没有可选的哈希，因为它是可选的，所以没关系）Rails辅助方法常常以这种方式使用可选参数，我们可以随意添加HTML属性，不需要Rails来处理。

在div中第二个内容是一列导航链接，使用未排序得列表标签**ul**，和列表项标签**li**：

```erb
<nav>
  <ul class="nav navbar-nav navbar-right">
    <li><%= link_to "Home",   '#' %></li>
    <li><%= link_to "Help",   '#' %></li>
    <li><%= link_to "Log in", '#' %></li>
  </ul>
</nav>

```

**<nav>**标签，尽管这里不需要正式地、更清晰地表明导航链接的目的。同时，标签**ul**上的类**nav，navbar-nav，navbar-right**类对Bootstrap有
特殊的意义。当我们在5.1.2节时包含Bootstrap CSS框架的时候，它们就会自动格式化HTML文件。正如你可以通过浏览器来确认效果一样，一旦Rails处理了布局文件，计算出内嵌Ruby代码的值，最后呈现在用户面前的列表看起来如下：

```html
<nav>
  <ul class="nav navbar-nav navbar-right">
    <li><a href="#">Home</a></li>
    <li><a href="#">Help</a></li>
    <li><a href="#">Log in</a></li>
  </ul>
</nav>
```

这个文本将是返回到浏览器的文本。

布局文件的最后部分是主要内容的**div**：

```erb
<div class="container">
  <%= yield %>
</div>
```

如同之前的，**container**类对Bootstrap来说有特殊的意义。正如我们在3.4.3节里学过的一样，**yield**方法把每页的内容插入网站布局文件。

除去网站的脚，我们将在5.1.3里添加，我们的布局文件现在完全了，我们通过访问主页看看结果。为了利用即将到来的样式元素，我们将给**home.html.erb**视图加一些额外的元素（代码清单5.2）。

```ruby
代码清单 5.2： 包含注册链接的主页。
# app/views/static_pages/home.html.erb
 <div class="center jumbotron">
  <h1>Welcome to the Sample App</h1>

  <h2>
    This is the home page for the
    <a href="http://www.railstutorial.org/">Ruby on Rails Tutorial</a>
    sample application.
  </h2>

  <%= link_to "Sign up now!", '#', class: "btn btn-lg btn-primary" %>
</div>

<%= link_to image_tag("rails.png", alt: "Rails logo"),
            'http://rubyonrails.org/' %>
```

为了在第七章中为我们的网站添加用户做准备，第一个**link_to**创建了一个只向表格的临时链接。
```ruby
<a href="#" class="btn btn-lg btn-primary">Sign up now!</a>
```

在**div**标签里，**jumbotron** CSS类对Bootstrap有特殊的意义，在注册页面里的按钮如**btn，btn-lg，btn-primary**类也一样。

第二个**link_to**炫耀了一下**image_tag**辅助方法，它以图片路径和可选哈希为参数，这里符号:alt设置了图片标签的**alt**属性。
为了让这段代码工作，需要一个名为**rails.png**的图片，你应该从Ruby on Rails主页，URL地址为http://rubyonrails.org/images/rails.png处下载，
然后放进**app/assets/images/**目录。假如你使用的是云IDE，或者其他和Unix类似的系统，你可以用**curl**工具：
```terminal
$ curl -O http://rubyonrails.org/images/rails.png
$ mv rails.png app/assets/images/
```

假如第二个命令失败了，有时会在云IDE失败，我也不太清楚原因，再试试第一条**curl**命令，直到文件正确地下载下来。（如要了解更多**curl**的知识，查看[征服命令行的第三章](http://conqueringthecommandline.com/book/curl)）因为我们在代码清单5.2里使用了**image_tag**辅助方法，Rails将
会使用asset pipeline自动在**app/assets/images/**目录查找（5.2节）。

为了让你更明白**image_tag**的作用，让我们看看生成的HTML文件：
```erb
<img alt="Rails logo" src="/assets/rails-9308b8f92fea4c19a3a0d8385b494526.png"/>
```

这里字符串9308b8f92fea4c19a3a0d8385b494526(你系统上的会不一样）是Rails为了确保唯一的文件名添加的，当它们被更新了以后，它会引起浏览器正确
地加载页面（而不是从浏览器缓存读取）。注意到**src**属性不包括**images**，而是使用了保存所有的资源（图片，Javascript，CSS）**assets**目录。在服务器上，Rails使用正确的app/assets/images/目录关联assets目录里的图片，但是只要浏览器觉得所有静态资源在同样的目录的话，这些静态资源就会较快的被加载。同时，alt属性的作用是假如网页从一个不能显示图片的程序进入（例如为视障服务的屏幕阅读器），就会显示。

现在，我们终于准备好看我们的劳动成果了，如图5.2。你说很让人腻味，对吧？可能如此。不过还是很开心，我们给HTML元素定义了实用的类，把我们置于可以使用CSS给我们网站添加样式的高度了。

![图5.2：没有自定义CSS的主页](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/layout_no_logo_or_custom_css_bootstrap_3rd_edition.png)

### 5.1.2 Bootstrap和自定义CSS

在5.1.1节，我们把许多HTML标签和CSS连接起来，它给我们在依据CSS构建布局增加了可观的灵活性。如5.1.1节提到的，这些类许多是针对Bootstrap的，
一个来自于Twitter的框架，使得制作好看的HTML5网页设计和UI变得很容易。在这节，我们将Bootstrap和自定义CSS规则组合起来，开始给示例程序添加一
些样式。使用Bootstrap使我们的网站可以自适应各种大小的屏幕，确保它在各种设备上的显示效果妥妥的。

我们要做的第一步是添加Bootstrap，在Rails里可以通过添加bootstrap-sass gem完成，如代码清单5.3显示的那样。Bootstrap框架原本为制作动态的样式表使用Less CSS语言，但是Rails asset pipeline默认支持的是（很相似）Sass语言（5.2节），所以bootstrap-sass把Less转换成Sass，
这样我们的应用程序就可以用所有Bootstrap的功能。

```ruby
代码清单 5.3： 把名为'bootstrap-sass'的Gem加入Gemfile。
source 'https://rubygems.org'

gem 'rails',                '5.0.0'
gem 'bootstrap-sass',       '3.2.0'
.
.
.
```

为了安装Bootstrap，我们和通常一样行bundle install：
```terminal
$ bundle install
```
尽管Rails generate自动为每个控制器都创建了单独的CSS，但是按照正确的顺序把它包含进来令人吃惊的困难，所以为了简化，我会把本书需要的所有CSS文件放在一个文件里。让自定义CSS工作的第一步是创建自定义CSS文件：

```ruby
$ touch app/assets/stylesheets/custom.css.scss
```
(这里使用了3.3.3节使用过的技巧，但是你可以用你喜欢的方法创建文件）这里目录名和文件名都很重要。目录
```ruby
# app/assets/stylesheets/
```
是asset pipeline（5.2节）的一部分，任何这个目录的样式表文件将自动地作为application.css的一部分包含进网站的布局文件application.html.erb中
。进一步，文件名custom.css.scss包含.css扩展，表明是一个CSS文件，.scss扩展，表明是“Sass CSS”文件，然后为asset pipeline使用Sass处理文件做
好准备。（我们从5.2.2节才会开始使用Sass，但是要让bootstrap-sass gem的魔法工作是需要的。）

在自定义CSS文件里，我们可以使用@import函数来包含Bootstrap（和关联的Sprockets工具一起），如同代码清单5.4显示的一样。
```ruby
代码清单 5.4： 添加Bootstrap CSS。
# app/assets/stylesheets/custom.css.scss
@import "bootstrap-sprockets";
@import "bootstrap";
```
在代码清单5.4里的两行，包含了整个Bootstrap CSS框架。重启网页服务器后，把变化插入开发的应用程序（通过按Ctrl-C，然后运行rails server如1.3.2节一样），结果Ruby图5.3所示。文本的位置不好，而且标志没有任何样式，但是颜色和注册按钮看上去不错。

![图5.3：使用Bootstrap CSS后的示例应用程序](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/sample_app_only_bootstrap_3rd_edition.png)

接下来，我们将加一些CSS，将会在整个网站里使用，用来样式化布局和每个网页，Ruby代码清单5.5里显示的一样。结果如图5.4.（在代码清单5.5里有很
少的CSS规则，为了对CSS规则有个直观的感受，使用CSS注释他们常会有所帮助，例如，通过把CSS规则放在**/* .. */**里，看看发生了什么）

```ruby
代码清单 5.5： 添加应用到所有页面的CSS。
# app/assets/stylesheets/custom.css.scss
@import "bootstrap-sprockets";
@import "bootstrap";

/* universal */

body {
  padding-top: 60px;
}

section {
  overflow: auto;
}

textarea {
  resize: vertical;
}

.center {
  text-align: center;
}

.center h1 {
  margin-bottom: 10px;
}
```
![图5.4：添加一些空格和其他全局样式](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/sample_app_universal_3rd_edition.png)

注意在代码清单5.5里的CSS有一个不变的表格。通常来说，CSS规则提及类，id，和HTML标签，或者它们的组合，后面跟着一列样式命令。例如，

```css
body {
  padding-top: 60px;
}
```
在页面顶部保留了60像素的内边距。因为在**header**标签里的**navbar-fixed-top**类，Bootstrap修补了导航栏到页面顶部的距离，所以内边距服务于
分隔导航和主要文本。（因为默认的导航栏颜色从Bootstrap 2.0以后改了，我们需要使用**navbar-inverse**类来让它变得黑色，而不是淡色。）同时，
规则里的CSS
```css
.center {
  text-align: center;
}
```
用**text-align: center**属性和**center**类相联系。（正如我们在代码清单5.7里看到的，井号#代表CSS定义的id）这意味着在任何HTML标签里（
例如**div**），如包含有类**center**，文本就会放置在页面的中间（我们会在代码清单5.2里看到使用这个类的例子）。

尽管Bootstrap应用CSS规则做出了很好的版面设计，我们也会为我们网站文本的显示添加一些自定义规则，如代码清单5.6所示。（不是所有的这些规则都
应用到主页，但是这里的每个规则都会在示例程序的某点上使用。）代码清单5.6的结果如图5.5所示。

```ruby
代码清单 5.6：添加美化页面的CSS规则
# app/assets/stylesheets/custom.css.scss
@import "bootstrap-sprockets";
@import "bootstrap";
.
.
.
/* typography */

h1, h2, h3, h4, h5, h6 {
  line-height: 1;
}

h1 {
  font-size: 3em;
  letter-spacing: -2px;
  margin-bottom: 30px;
  text-align: center;
}

h2 {
  font-size: 1.2em;
  letter-spacing: -1px;
  margin-bottom: 30px;
  text-align: center;
  font-weight: normal;
  color: #777;
}

p {
  font-size: 1.1em;
  line-height: 1.7em;
}
```

![图5.5：添加一些版面设计](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/sample_app_typography_3rd_edition.png)

最后，我们将给网站的标志加一些样式，虽然它只包含文本“sample app"。在代码清单5.7里，CSS将文本转化成大写，然后修改尺寸，颜色，和位置。（我
们使用CSS id，因为我们想要网站标志在页面上仅显示一次，但是你也可以使用类代替。）

```css
代码清单 5.7： 为网站的Logo添加CSS。
# app/assets/stylesheets/custom.css.scss
@import "bootstrap-sprockets";
@import "bootstrap";
.
.
.
/* header */

#logo {
  float: left;
  margin-right: 10px;
  font-size: 1.7em;
  color: #fff;
  text-transform: uppercase;
  letter-spacing: -1px;
  padding-top: 9px;
  font-weight: bold;
}

#logo:hover {
  color: #fff;
  text-decoration: none;
}

```

这里，**color: #fff**，将网站标志的颜色转为白色。HTML颜色可以通过三组16进制的数字表示，依次是红，绿蓝。代码**#ffffff**最大化三种颜色，生
成纯白色，**#fff**是**ffffff**的简写。CSS标准也定义了许多HTML颜色的同义词，包括**white**为**#fff**。
代码清单5.7里面CSS的效果Ruby图5.6显示。

![图5.6：美化了的样式示例程序](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/sample_app_logo_3rd_edition.png)

### 5.1.3 片段（Partial）

尽管在代码清单5.1里的布局文件达到了它的目的，不过看起来有点乱。HTML shim用了三行，而且使用了IE专有的语法，所以把它移动到单独的文件会更好。另外，HTML头部形成了一个逻辑单元，所以所有的这些应该放在一处。Rails中通过使用一个叫partial的工具实现这些。让我们先看看使用partial后的
布局文件（代码清单5.8）。

```html
<!DOCTYPE html>
<html>
  <head>
    <title><%= full_title(yield(:title)) %></title>
    <%= stylesheet_link_tag 'application', media: 'all',
                                           'data-turbolinks-track' => true %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
    <%= csrf_meta_tags %>
    <%= render 'layouts/shim' %>
  </head>
  <body>
    <%= render 'layouts/header' %>
    <div class="container">
      <%= yield %>
    </div>
  </body>
</html>
```
在代码清单5.8里，我们将替换HTML shim样式行用Rails中名为**render**的辅助方法：
```ruby
<%= render 'layouts/shim' %>
```
这行的效果是寻找名为**app/views/layouts/_shim.html.erb**的文件，计算它的内容，然后插入视图。（回忆<%= .. %>需要计算Ruby表达式，然后把结
果插入模板的内嵌Ruby语法。）注意文件**_shim.html.erb**名前面的下划线，这个下划线是命名partial统一的惯例，所以在目录里撇一眼就能识
别出一个目录里所有的partial。

当然，为了让partial工作，我们不得不创建相应的文件，然后填入内容。在shim片段的例子中，就是代码清单5.1里的三行代码，结果如代码清单5.9所示：
```html
代码清单 5.9： HTML shim Partial文件
# app/views/layouts/_shim.html.erb
<!--[if lt IE 9]>
  <script src="//cdnjs.cloudflare.com/ajax/libs/html5shiv/r29/html5.min.js">
  </script>
<![endif]-->
```

相似地，我们可以把头部的内容移入片段，如代码清单5.10所示，然后用**render**方法把它插入布局文件。（对于Partial文件，通常来说你只能使用文本编辑器手动创建）。

```html
代码清单 5.10： 网站头部的Partial
# app/views/layouts/_header.html.erb
<header class="navbar navbar-fixed-top navbar-inverse">
  <div class="container">
    <%= link_to "sample app", '#', id: "logo" %>
    <nav>
      <ul class="nav navbar-nav navbar-right">
        <li><%= link_to "Home",   '#' %></li>
        <li><%= link_to "Help",   '#' %></li>
        <li><%= link_to "Log in", '#' %></li>
      </ul>
    </nav>
  </div>
</header>
```

现在我们知道怎么创建片段，让我们和头部一起，给站点添加底部。我知道现在你可能已经猜出来我们会给它起什么名字了，是的，就是**_footer.html.erb**，然后把它放到布局目录里（代码清单5.11）。

```html
代码清单 5.11：网站底部的Partial
# app/views/layouts/_footer.html.erb
<footer class="footer">
  <small>
    The <a href="http://www.railstutorial.org/">Ruby on Rails Tutorial</a>
    by <a href="http://www.michaelhartl.com/">Michael Hartl</a>
  </small>
  <nav>
    <ul>
      <li><%= link_to "About",   '#' %></li>
      <li><%= link_to "Contact", '#' %></li>
      <li><a href="http://news.railstutorial.org/">News</a></li>
    </ul>
  </nav>
</footer>
```
和头部一样，在底部我们使用**link_to**链接“关于”和“联系”页面，现在用‘#’。（和**header**一样，**footer**也是HTML5新加的）

我们可以通过和样式表和头部Partial一样的模式的在布局文件里渲染底部Partial，（代码清单5.12）

```html
代码清单 5.12：使用了底部Partial的网站布局文件。
# app/views/layouts/application.html.erb
 <!DOCTYPE html>
<html>
  <head>
    <title><%= full_title(yield(:title)) %></title>
    <%= stylesheet_link_tag "application", media: "all",
                                           "data-turbolinks-track" => true %>
    <%= javascript_include_tag "application", "data-turbolinks-track" => true %>
    <%= csrf_meta_tags %>
    <%= render 'layouts/shim' %>
  </head>
  <body>
    <%= render 'layouts/header' %>
    <div class="container">
      <%= yield %>
      <%= render 'layouts/footer' %>
    </div>
  </body>
</html>
```

当然，因为底部还没有样式，所以自然有点丑陋（代码清单5.13）。效果如图5.7所示。

```scss
代码清单 5.13： 为网站底部添加CSS样式。
# app/assets/stylesheets/custom.css.scss
 .
.
.
/* footer */

footer {
  margin-top: 45px;
  padding-top: 5px;
  border-top: 1px solid #eaeaea;
  color: #777;
}

footer a {
  color: #555;
}

footer a:hover {
  color: #222;
}

footer small {
  float: left;
}

footer ul {
  float: right;
  list-style: none;
}

footer ul li {
  float: left;
  margin-left: 15px;
}
```

![图5.7：添加了底部的主页](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/site_with_footer_bootstrap_3rd_edition.png)

## 5.2 Sass和资源管线(asset pipline)
最近的Rails版本值得额外注意的是资源管线（asset pipeline），它显著地提高了静态资源，例如CSS、Javascript和图片资源的产生和管理。这节首先给我们一个资源管线的高度概述，然后显示怎样使用Sass--一个编写CSS强有力的工具。

### 5.2.1 资源管线
资源管线加入了许多Rails帽子下的特性，但是从典型的Rails开发者的角度来审视的话，主要有三个特性需要理解：资产目录、说明文件、预处理引擎。让我们依次说明。

### 资产目录
在Rails 3以及之前的版本，静态资产位于**public**目录，如下：

* public/stylesheets
* public/javascripts
* public/images

在这些目录里的文件（甚至在3.0以后）自动通过到http://www.example.com/stylesheet等等的请求服务。

在最近的Rails版本，为静态资源提供了三个显而易见的目录，每一个有它自己的目的：
* app/assets: 目前应用程序专用的资源
* lib/assets：你的开发团队写的库的资源
* vendor/assets： 来自第三方的资源

你可能猜到了，这些目录每个都有三个子目录，如：
```terminal
$ ls app/assets/
# images/  javascripts/  stylesheets/
```
到了现在，我们到了理解5.1.2节里自定义CSS文件位置背后隐藏的动机的时候了：**custom.css.scss**是范例程序专用的，所以把它放到**app/assets/stylesheets**。

### 清单文件

一旦你把资产放在了它们的逻辑位置，你可以使用清单文件告诉Rails（通过[Sprockets](https://github.com/sstephenson/sprockets) gem）怎样把它们组合成一个文件。（这是针对CSS和Javascript文件而言，不适用于图片资源）。正如例子所示，让我们看看范例应用程序的默认的CSS文件的代码清单（代码清单5.14）

```css
/*
 * This is a manifest file that'll be compiled into application.css, which
 * will include all the files listed below.
 *
 * Any CSS and SCSS file within this directory, lib/assets/stylesheets,
 * vendor/assets/stylesheets, or vendor/assets/stylesheets of plugins, if any,
 * can be referenced here using a relative path.
 *
 * You're free to add application-wide styles to this file and they'll appear
 * at the bottom of the compiled file so the styles you add here take
 * precedence over styles defined in any styles defined in the other CSS/SCSS
 * files in this directory. It is generally better to create a new file per
 * style scope.
 *
 *= require_tree .
 *= require_self
 */
```

关键的行实际上是CSS注释，但是Sprockets用它来包含正确的文件：

```css
/*
 .
 .
 .
 *= require_tree .
 *= require_self
*/
```

这里
```css
*= require_tree .
```
确保所有的在**app/assets/stylesheets**目录里的CSS文件（包含树子目录）被包含进入应用程序的CSS文件。这行
```css
*=require_self
```
在加载后续CSS文件时，把自身也包含进来。

Rails创建了合理的默认文件，在本书里我们不需要做任何修改，但是假如你需要修改它，[Rails指南里关于资源管线](http://guides.rubyonrails.org/asset_pipeline.html）有更多详细信息。

### 预处理引擎
你放置好网站的静态资源以后，Rails会通过运行几个预处理引擎和使用清单文件把它们组合起来，为输出到浏览器做准备。我们通过文件扩展名告诉Rails使用那个处理器处理，最常用的三个是Sass的**.scss**，Coffeescript的**.coffee**和内嵌Ruby的**.erb**(ERb)。我们在3.4.3节第一次介绍了ERb的知识，在5.2.2节介绍了Sass的知识。在本书中我们不需要CoffeeScript，它是一个可以编译成Javascript的小语言。（[RailsCast](http://railscasts.com/episodes/267-coffeescript-basics)是学习CoffeeScript的好地方。）

预处理引擎可以链接起来，所以foobar.js.coffee会运行CoffeeScript处理器，foobar.js.erb.coffee会依次运行CoffeeScript和ERb（按照从左到右的顺序运行，例如，先运行CoffeeScript）

###生产中的效率问题

资源管线最好的东西之一是它会自动在生产环境的应用程序里优化资源。CSS和Javascript文件组织的传统方法是将按照功能将代码分别存放到不同的文件中，在这些文件中使用漂亮的代码格式（通过代码缩进）。虽然对程序员来说比较方便，但是会导致程序在生产环境中效率低下。加载多个文件会显著地拖慢页面加载的时间，这是影响网络用户体验的最主要因素之一。有了资源管线，我们不再需要在速度和便利之间作出选择：我们可以在开发环境使用几个格式化好的文件，然后在生产环境下通过资源管线实现文件有效加载。具体来说，资源管线会把应用程序所有的CSS文件汇总至（application.css），把所有的Javascript程序合并至javascript文件（application.js)，然后移除文件内不必要的空格和缩进以及其他影响资源文件大小的因素，再通过压缩合并后的文件，进一步减小文件体积。结果是最好的两个世界：程序员感觉友好的开发环境和用户体验很好的生产环境。

### 5.2.2 句法超赞的CSS文件

Sass是为了编写CSS而发明的语言，在许多方面提高了CSS。本节我们学习最重要的两个语法提升：嵌套和变量。（另一个技术，混入（mixin），将在7.1.1节介绍）。

如在5.1.2节简单介绍的一样，Sass是名为SCSS的格式（用**.scss**文件扩展名表示），它是CSS的严格的超集。即，SCSS仅仅是为CSS增强了一些特性，而不是重新定义新的语法。这意味着每个有效的CSS文件也是有效的SCSS文件，对于
已有的CSS的项目文件很方便。在我们的例子中，我们使用SCSS是为了使用Bootstrap。因为Rails资源管线会自动使用Sass处理**.scss**扩展，所以应用程序先运行Sass处理器将**custome.css.scss**文件处理成标准的CSS文件，然后打包发送到浏览器。

#### 嵌套
样式表经常会嵌套定义元素的样式，例如，在代码清单5.5中，我们有两个规则，**.center**和**.center h1**：

```css
.center {
  text-align: center;
}

.center h1 {
  margin-bottom: 10px;
}

```
我们可以用Sass替换为
```css
.center {
  text-align: center;
  h1 {
    margin-bottom: 10px;
  }
}

```

这里嵌套的**h1**规则自动继承了**.center**上下文。

另一种嵌套语法有一点点不同。在代码清单5.7里，我们有以下代码：

```css
#logo {
  float: left;
  margin-right: 10px;
  font-size: 1.7em;
  color: #fff;
  text-transform: uppercase;
  letter-spacing: -1px;
  padding-top: 9px;
  font-weight: bold;
}

#logo:hover {
  color: #fff;
  text-decoration: none;
}

```

这里CSS ID “#logo”显示了两次，一次是单独出现、一次是和**hover**属性一起出现（当鼠标悬在问题里的要素控制它的表现）。为了嵌套第二个样式，我们需要引用父级元素**#logo**；在SCSS里，用连接符“&”：
```css
#logo {
  float: left;
  margin-right: 10px;
  font-size: 1.7em;
  color: #fff;
  text-transform: uppercase;
  letter-spacing: -1px;
  padding-top: 9px;
  font-weight: bold;
  &:hover {
    color: #fff;
    text-decoration: none;
  }
}

```

Sass把**&:hover**编译为**#logo:hover**，作为从SCSS转化为CSS的一部分。

把这两种嵌套技术应用到代码清单5.13中footer的CSS代码中，它变成：
```css
footer {
  margin-top: 45px;
  padding-top: 5px;
  border-top: 1px solid #eaeaea;
  color: #777;
  a {
    color: #555;
    &:hover {
      color: #222;
    }
  }
  small {
    float: left;
  }
  ul {
    float: right;
    list-style: none;
    li {
      float: left;
      margin-left: 15px;
    }
  }
}

```

手动转换一下代码清单5.13是掌握Sass语法的好方法，你应该确认转化后的CSS仍然工作正常。

### 变量

Sass允许我们通过定义变量来写出更多富有表达力的代码。例如，在代码清单5.6和5.13中，我们看见有重复的颜色代码：
```css
h2 {
  .
  .
  .
  color: #777;
}
.
.
.
footer {
  .
  .
  .
  color: #777;
}


```
在这里，**#777**是浅灰，然后我们可以通过定义变量，给它一个名字，如下：
```css
$light-gray: #777;

```

允许我们像这样一样重写SCSS：

```css
$light-gray: #777;
.
.
.
h2 {
  .
  .
  .
  color: $light-gray;
}
.
.
.
footer {
  .
  .
  .
  color: $light-gray;
}


```

把Sass嵌套和变量定义应用到整个SCSS文件里，最后的文件如代码清单5.15所示。这里使用了Sass变量（参考Bootstrap
Less变量定义）和内建的已命名的颜色（如，white为#fff）。通过这些手段**footer**标签里的样式文件可读性有了显著的提高。

```css
代码清单 5.15：使用嵌套和变量的SCSS文件。
# app/assets/stylesheets/custom.css.scss
@import "bootstrap-sprockets";
@import "bootstrap";

/* mixins, variables, etc. */

$gray-medium-light: #eaeaea;

/* universal */

body {
  padding-top: 60px;
}

section {
  overflow: auto;
}

textarea {
  resize: vertical;
}

.center {
  text-align: center;
  h1 {
    margin-bottom: 10px;
  }
}

/* typography */

h1, h2, h3, h4, h5, h6 {
  line-height: 1;
}

h1 {
  font-size: 3em;
  letter-spacing: -2px;
  margin-bottom: 30px;
  text-align: center;
}

h2 {
  font-size: 1.2em;
  letter-spacing: -1px;
  margin-bottom: 30px;
  text-align: center;
  font-weight: normal;
  color: $gray-light;
}

p {
  font-size: 1.1em;
  line-height: 1.7em;
}


/* header */

#logo {
  float: left;
  margin-right: 10px;
  font-size: 1.7em;
  color: white;
  text-transform: uppercase;
  letter-spacing: -1px;
  padding-top: 9px;
  font-weight: bold;
  &:hover {
    color: white;
    text-decoration: none;
  }
}

/* footer */

footer {
  margin-top: 45px;
  padding-top: 5px;
  border-top: 1px solid $gray-medium-light;
  color: $gray-light;
  a {
    color: $gray;
    &:hover {
      color: $gray-darker;
    }
  }
  small {
    float: left;
  }
  ul {
    float: right;
    list-style: none;
    li {
      float: left;
      margin-left: 15px;
    }
  }
}


```

Sass为我们提供了甚至更多的方法去简化我们的样式表，但是代码清单5.15里的使用的最重要的特性，给我们开了个好头。你可以去[Sass官网](http://sass-lang.com/)查看更多细节。

## 5.3 布局链接
既然我们已经为网站的布局定义了得体的样式，是时候开始用真实的链接来替换占位符#代表的链接了。当然，我们可以用硬编码的方式实现：
```html
<a href="/static_pages/about">About</a>

```

但是这不是Rails的方式。一者，“关于”页面的URL是/about而不是/static_pages/about；再者，依据Rails的惯例会使用具名路由（named route)，代码看起来像：
```ruby
<%= link_to "About", about_path %>
```

这种方法的代码有更直白的意思，而且富有弹性。因为我们可以通过改变**about_path**的定义即可实现所有使用**about_path**的URL都被改变。

下表是我们规划的链接列表，如表5.1所示，和它们映射的URL、路由一起。我们在3.4.4节认真看过第一个路由，我们在这章结束将实现除最后一个以外的所有路由（我们在第八章创建最后一个路由）。

Page | URL | Namedroute
----|----|----
Home |/ |root_path
About |/about_path|about_path
Help|/help|help_path
Contact|/contact|contact_path
Sign up|/signup|signup_path
Log in|/login|login_path

表5.1： 路由和网站链接映射的URL

### 5.3.1 “联系”页面
为了补全网站信息，我们再添加一个“联系（Contact）”页面，我们在第三章的时候曾布置过这个作业。测试代码如代码清单5.16，它简单的模仿了代码清单3.22里内容。

```ruby
代码清单 5.16： Contact的页面的测试。红色
# test/controllers/static_pages_controller_test.rb
require 'test_helper'

class StaticPagesControllerTest < ActionController::TestCase

  test "should get home" do
    get static_pages_home_url
    assert_response :success
    assert_select "title", "Ruby on Rails Tutorial Sample App"
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

  test "should get contact" do
    get static_pages_contact_url
    assert_response :success
    assert_select "title", "Contact | Ruby on Rails Tutorial Sample App"
  end
end
```

到这点，在代码清单5.16里的测试应该是红色的：

```terminal
$ bundle exec rails test
```
应用程序的代码几乎和3.3节的“关于”页面差不多：首先我们更新了路由（代码清单5.18），然后我们给静态页面控制器加了一个**contact**动作，最后我们创建了一个“联系”视图（代码清单5.20）。

```ruby
代码清单 5.18： 添加了到“联系（Contact）”页面的路由。红色
# config/routes.rb
Rails.application.routes.draw do
  root 'static_pages#home'
  get  'static_pages/help'
  get  'static_pages/about'
  get  'static_pages/contact'
end
```

```ruby
代码清单 5.19：在控制器中添加了“Contact”动作。红色
# app/controllers/static_pages_controller.rb
class StaticPagesController < ApplicationController
  .
  .
  .
  def contact
  end
end
```

```ruby
代码清单 5.20： 为Contact页面添加视图。绿色
# app/views/static_pages/contact.html.erb
<% provide(:title, 'Contact') %>
<h1>Contact</h1>
<p>
  Contact the Ruby on Rails Tutorial about the sample app at the
  <a href="http://www.railstutorial.org/#contact">contact page</a>.
</p>
```

现在确保测试是绿色的：
```terminal
代码清单 5.21： 绿色
$ bundle exec rails test
```

### 5.3.2 Rails 路由
为了给Sample App的静态页面添加具名路由，我们通过编辑路由文件**config/routes.rb**来实现，Rails使用它来定义URL映射。我们通过回顾主页的路由（在3.4.4节定义的）开始，它是特殊的例子，然后为其余的静态页面定义一套路由。

到目前为止，我们已经见过了三个定义根路由的例子，从Hello应用开始（代码清单1.10）代码：
```ruby
root ‘application#hello’
```
玩具应用代码(代码清单2.3）
```ruby
root 'users#index'
```
和Sample APP程序代码（代码清单3.37）
```ruby
root 'static_pages#home'
```
在每个例子中，**root**方法为根路径“/”路由到控制器和指定的动作建立了映射。这样定义跟路由有另一个重要的影响，就是创建了具名路由，允许我们通过名字而不是原始的URL查找路由。在这里，形成了两个路由，分别是**root_path**和**root_url**，不同之处在于后面的路由包含整个URL：

```
root_path -> '/'
root_url  -> 'http://www.example.com/'
```

在本书中，我们将遵循一般的惯例，使用_path形式，除了重定向的时候我们使用_url形式。（这是因为HTTP标准的技术重定向后需要全URL，尽管在大部分浏览器两种方法都工作。）

为了为“帮助”、“关于”和“联系”页面定义具名路由，我们需要改变代码清单5.18的get规则，把：
```ruby
get 'static_pages/help'
```
改为
```ruby
get ‘help’ => 'static_pages#help'
```
这些模式的第二个路由GET请求URL /help到静态页面控制器里面help动作，以便我们可以使用URL /help代替啰嗦的/static_pages/help。和根路由一样，这创建两个具名路由，help_path和help_url:

```ruby
help_path -> '/help'
help_url  -> 'http://www.example.com/help'
```

应用这个规则到剩下的静态页面路由，从代码清单5.18转换为代码清单5.22.

```ruby
代码清单 5.22： Routes for static pages.
# config/routes.rb
 Rails.application.routes.draw do
  root             'static_pages#home'
  get 'help'    => 'static_pages#help'
  get 'about'   => 'static_pages#about'
  get 'contact' => 'static_pages#contact'
end
```

### 5.3.3 使用具名路由

使用代码清单5.22里定义的路由，该是我们在网站布局文件里使用具名路由的时候了。只是需要简单的把正确的路由填充到第二个参数。例如，我们把
```ruby
<%= link_to "About", '#' %>
```
修改为
```ruby
<%= link_to "About", about_path %>
```
等等。

我们将从网站头部Partial文件_header.html.erb开始（代码清单5.23），这里有主页和帮助页面的链接。我们将遵循网页设计原理，把网站标志链接到主页。

```html
代码清单 5.23： 带链接的网站头部Partial文件。
# app/views/layouts/_header.html.erb
<header class="navbar navbar-fixed-top navbar-inverse">
  <div class="container">
    <%= link_to "sample app", root_path, id: "logo" %>
    <nav>
      <ul class="nav navbar-nav navbar-right">
        <li><%= link_to "Home",    root_path %></li>
        <li><%= link_to "Help",    help_path %></li>
        <li><%= link_to "Log in", '#' %></li>
      </ul>
    </nav>
  </div>
</header>
```

到第八章我们才会为“登陆”添加命名路由，所以我们现在仍然使用占位符“#”。

另一个有链接的地方是网站底部Partial，_footer.html.erb的“关于”和“联系”页面（代码清单5.24）

```ruby
代码清单 5.24： 带链接的网站底部Partial文件。
# app/views/layouts/_footer.html.erb
<footer class="footer">
  <small>
    The <a href="http://www.railstutorial.org/">Ruby on Rails Tutorial</a>
    by <a href="http://www.michaelhartl.com/">Michael Hartl</a>
  </small>
  <nav>
    <ul>
      <li><%= link_to "About",   about_path %></li>
      <li><%= link_to "Contact", contact_path %></li>
      <li><a href="http://news.railstutorial.org/">News</a></li>
    </ul>
  </nav>
</footer>
```
现在，我们的布局文件已经有了第三章创建的所有静态页面的链接，即，例如，/about映射到关于页面（图5.8）。

![图5.8：路径/about的关于页面](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/about_page_styled_3rd_edition.png)

### 5.3.4 布局文件链接测试

既然我们经用实际的链接替换了临时链接，测试一下这些链接，确保它们可以正确地工作是个好主意。我们可以通过手动通过浏览器来完成这个任务。首先访问根路径，然后手动检查一下每个链接，但是很快你就会变得麻烦。因此我们将使用集成测试来模拟同样的步骤。集成测试允许我们写端到端的应用程序行为测试。我们可以通过生成集成测试模板开始生成一个叫site_layout的测试:

```terminal
$ rails generate integration_test site_layout
  invoke test_unit
  create test/integration/site_layout_test.rb
```

注意Rails生成器自动给测试文件添加了“test”后缀。

我们为布局文件制定的测试计划是检查我们网站的HTML结构：
1.访问根路径（主页）
2.确认渲染正确的页面
3.检查主页、帮助页面、关于页面和联系页面链接正确

代码清单5.25显示了我们怎么通过使用Rails集成测试把这些步骤翻译称代码，用**assert_template**方法来确认主页渲染了正确的视图。为了使用**assert_template**我们需要先在Gemfile文件里添加**rails-controller_testing**
 Gem。
```ruby
# Gemfile
...

group :test do
  ...
  gem 'rails-controller-testing' #添加这两个GEM到:test组
  gem 'rails-dom-testing'
  ...
end

...
```
然后运行
```
$ bundle install
```
然后修改测试文件：
```ruby
代码清单 5.25： A test for the links on the layout. 绿色
# test/integration/site_layout_test.rb
 require 'test_helper'

class SiteLayoutTest < ActionDispatch::IntegrationTest

  test "layout links" do
    get root_path
    assert_template 'static_pages/home'
    assert_select "a[href=?]", root_path, count: 2
    assert_select "a[href=?]", help_path
    assert_select "a[href=?]", about_path
    assert_select "a[href=?]", contact_path
  end
end
```

代码清单5.25使用了一些高级的assert_select方法，之前在代码清单3.22和5.16里见过。在这个例子中，允许我们通过使用标签名a和属性herf测试指定链接组合，如
```ruby
assert_select "a[href=?]", about_path
```

这里，Rails自动插入about_path的值，代替问号（假如需要转义任何特殊字符），然后检查表格的HTML标签
```html
<a href="/about">...</a>
```

注意根路径的断言确认有两个这样的链接（一个是网站标志一个是导航菜单中的内容）：
```ruby
asset_select "a[href=?]", root_path, count: 2
```

这个断言确保代码清单5.23里定义的主页两个链接都存在。
更多的assert_select用法在表5.2里。assert_select是非常灵活，功能很强（比这里的选项还多）。经验显示通过仅仅测试HTML元素（例如网站布局文件链接）这样的轻度测试是明智的，不会为测试增加很多时间。

Code | Matching HTML
---|---
assert_select "div" | <div>foobar</div>
assert_select "div", "foobar" | <div>foobar</div>
assert_select "div.nav" | <div class="nav">foobar</div>
assert_select "div#profile" | <div id="profile">foobar</div>
assert_select "div[name=yo]" | <div name="yo">hey</div>
assert_select "a[href=?]", ’/’, count: 16 | <a href="/">foo</a>
assert_select "a[href=?]", ’/’, text: "foo" | <a href="/">foo</a> 

表5.2： 一些assert_select用法

为了确保代码清单5.25的测试可以通过，我们可以仅运行集成测试，使用下面的命令：
```terminal
代码清单 5.26： 绿色
$ bundle exec rails test:integration
```
假如一切正常，你应该运行完整测试集来确认所有的测试都是绿色的：
```terminal
代码清单 5.27： 绿色
$ bundle exec rails test
```
随着为布局文件链接加了集成测试，我们该是使用测试来快速捕捉回溯了。

## 5.4 用户登陆： 第一步
作为我们在布局文件和路由的最难处，这节我们将为登陆页面创建一个路由，然后接着创建第二个控制器。这是允许用户注册我们网站的重要一步，接下来模块化用户。在第六章，我们将完成第七章里的一部分任务。

### 5.4.1 用户控制器
在3.2节中我们创建了第一个控制器--静态页面控制器(StaticPagesController)。是时候创建另一个了--用户控制器（UsersController）。和之前一样，我们使用generate来创建一个简单的控制器，为新用户提供注册页面。遵循Rails偏爱的REST架构的惯例，我们命名创建新用户的动作为new，我们可以通过传递new作为generate的参数来实现。结果如代码清单5.28所示。
```terminal
代码清单 5.28：生成Users控制器（包含一个动作：new）。
$ rails generate controller Users new
      create  app/controllers/users_controller.rb
       route  get 'users/new'
      invoke  erb
      create    app/views/users
      create    app/views/users/new.html.erb
      invoke  test_unit
      create    test/controllers/users_controller_test.rb
      invoke  helper
      create    app/helpers/users_helper.rb
      invoke    test_unit
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/users.js.coffee
      invoke    scss
      create      app/assets/stylesheets/users.css.scss
```
如同我们想要的结果，代码清单5.28创建了一个用户控制器（UsersController），并为这个控制器添加了动作new，还创建了一个用户视图（app/views/users/new.html.erb，详见代码清单5.31）。它也为新用户页面创建了一个最小化的测试（代码清单5.32），现在测试应该通过：

```terminal
代码清单 5.29： 绿色
$ bundle exec rails test
```
```ruby
代码清单 5.30： 初始化的UsersController，包含名为new的动作。
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  def new
  end
end
```
```ruby
代码清单 5.31：初始化用户控制器的new视图。
# app/views/users/new.html.erb
<h1>Users#new</h1>
<p>Find me in app/views/users/new.html.erb</p>
```

```ruby
代码清单 5.32：控制器测试视图。绿色
# test/controllers/users_controller_test.rb
require 'test_helper'

class UsersControllerTest < ActionController::TestCase

  test "should get new" do
    get users_new_url
    assert_response :success
  end
end
```

### 5.4.2 注册URL
有了5.4.1节里的代码，新用户就可以通过/users/new页面在我们的网站注册了。但是回忆表5.1，我们想要用户通过/signup来注册。我们模仿代码清单5.22里的例子，为用户注册地址添加 **get ‘/signup’**的路由，如代码清单5.33所示。
```ruby
代码清单 5.33：注册页面的路由。
# config/routes.rb
Rails.application.routes.draw do
  root             'static_pages#home'
  get 'help'    => 'static_pages#help'
  get 'about'   => 'static_pages#about'
  get 'contact' => 'static_pages#contact'
  get 'signup'  => 'users#new'
end
```

接下来，我们使用新定义的具名路由把正确的链接按钮添加到主页。和其他路由一样，get 'signup'自动为我们生成了两个具名路由signup_path和signup_url。
我们把它放到代码清单5.34里，为注册页面添加测试这个任务当做练习（5.6节）留给你来完成。

```ruby
代码清单 5.34：为按钮添加“注册”链接。
# app/views/static_pages/home.html.erb
<div class="center jumbotron">
  <h1>Welcome to the Sample App</h1>

  <h2>
    This is the home page for the
    <a href="http://www.railstutorial.org/">Ruby on Rails Tutorial</a>
    sample application.
  </h2>

  <%= link_to "Sign up now!", signup_path, class: "btn btn-lg btn-primary" %>
</div>

<%= link_to image_tag("rails.png", alt: "Rails logo"),
            'http://rubyonrails.org/' %>
```

最后，我们再为注册页面添加一个自定义临时视图（代码清单5.35）。

```erb
代码清单 5.35：初始化临时注册视图。
# app/views/users/new.html.erb
<% provide(:title, 'Sign up') %>
<h1>Sign up</h1>
<p>This will be a signup page for new users.</p>
```

有了这个视图，我们完成了链接和命名路由，指定我们将在第八章会添加得登陆路由。新用户注册页面的结果（URL地址/signup）显示如图5.9。

![图5.9：在/signup的新注册页](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/new_signup_page_3rd_edition.png)

## 5.5 结论

在这章，我们把应用程序布局文件锤炼成形，然后润色了一下路由。本书剩余部分致力于充实应用程序的内容：
首先，实现用户注册、登陆和退出；接下来添加用户微博功能；然后，添加关注其他用户的能力。

到了这里，假如你使用Git，你应该把你的变化合并到主分支：

```terminal
$ bundle exec rails test
$ git add -A
$ git commit -m "Finish layout and routes"
$ git checkout master
$ git merge filling-in-layout
```
然后推送到Bitbucket
```terminal
$ git push
```
最后，部署到Heroku
```
$ git push heroku
```
部署的结果应该是在生产环境服务器工作Sample App程序（图5.10）。

![图5.10：生产环境里的Sample App程序](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/layout_production.png)

### 5.5.1 这章我们学到了什么
* 使用HTML5，我们可以用网站标志、头部、底部和主体内容定义网站的布局文件
* Rails片段(Partial)常用来为在单独的文件里放置模板提供便利
* CSS允许我们依据CSS类和id样式化站点布局
* Bootstrap框架使得快速开发一个漂亮的网站变得很容易
* Sass和资源管线允许我们消除我们CSS里的重复，然后为生产环境提供打包好的高效的资源文件
* Rails允许我们自定义路由规则，提供具名路由
* 集成测试高效地模拟浏览器在页面间转换


## 5.6 练习

1. 如5.2.2节建议的那样，学习代码清单5.13到5.15例子，手动把footer部分的CSS转换成SCSS。
2. 在代码清单5.25的集成测试里，添加代码使用get方法访问注册页面，然后确认结果页的标题是正确的。
3. 通过包含Application Helper，在测试中使用full_title辅助方法。Ruby代码清单5.36所示。然后使用Ruby代码清单5.37的代码（它是从前面的练习扩展的解决方案）测试标题是否正确。这个测试是易碎的，不过，因为现在任何在基础标题的错误（例如“Ruby on Rails Totoial”）不会被测试集捕获。
通过编写一个直接对full_title进行测试的辅助方法来解决这个问题。需要创建一个文件来测试应用程序辅助方法，然后在代码清单5.38里FILL_IN的地方填入正确的代码。（代码清单5.38用操作符==来确认assert_equal <想要的值>,<实际的值>，期待的结果和实际的值匹配）。

```ruby
代码清单 5.36：在测试中包含Application辅助方法。
# test/test_helper.rb
ENV['RAILS_ENV'] ||= 'test'
.
.
.
class ActiveSupport::TestCase
  fixtures :all
  include ApplicationHelper
  .
  .
  .
end
```

```ruby
代码清单 5.37：在测试中使用full_title辅助方法。 绿色
# test/integration/site_layout_test.rb
require 'test_helper'

class SiteLayoutTest < ActionDispatch::IntegrationTest

  test "layout links" do
    get root_path
    assert_template 'static_pages/home'
    assert_select "a[href=?]", root_path, count: 2
    assert_select "a[href=?]", help_path
    assert_select "a[href=?]", about_path
    assert_select "a[href=?]", contact_path
    get signup_path
    assert_select "title", full_title("Sign up")
  end
end
```
```ruby
代码清单 5.38：直接测试full_title辅助方法。
# test/helpers/application_helper_test.rb
require 'test_helper'

class ApplicationHelperTest < ActionView::TestCase
  test "full title helper" do
    assert_equal full_title,         FILL_IN
    assert_equal full_title("Help"), FILL_IN
  end
end
```
