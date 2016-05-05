# 第四章 Rails风味的Ruby
经过第三章中示例程序的练习，这章我们将深入探索Ruby编程语言中的一些对Rails来说比较重要的内容。Ruby是一个块头较大得语言，但是幸运的是成为一个有效的Rails程序员需要的Ruby知识的子集相对比较小。和其他通常介绍Ruby的材料有些不同，这章的目的是不管你之前有没有相关经验，都会让你在Rails风味的Ruby知识方面打下牢固的基础。它覆盖了许多知识，如果第一次不能掌握也没有关系，我会在未来的章节里经常引用。

## 4.1 动机
如同我们在上一章看到的一样，就算我们没有掌握必要的Ruby知识，开发一个Rails应用程序的骨架，甚至测试它都是可能的。我们是依赖教程和错误信息的提示做到了这些。可是这种情形不可能一直持续，从这章开始，我们将直接面对我们在Ruby知识方面的软肋。

看看我们之前新写的应用程序，只是使用Rails布局更新了我们以静态为主的页面，消除了我们视图里的重复。如代码清单4.1所示（和代码清单3.32一样）。

```erb
代码清单 4.1: sample_app网站布局文件。
# app/views/layouts/application.html.erb
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
让我们把焦点放在代码清单4.1里面的一行：
```erb
<%= stylesheet_link_tag 'application', media: 'all',
                                       'data-turbolinks-track' => true %>
```
这里使用了Rails内建的函数**stylesheet_link_tag**(你可以在[Rails
API](http://api.rubyonrails.org/classes/ActionView/Helpers/AssetTagHelper.html#method-i-stylesheet_link_tag)了解更多）包含为所有的[媒体类型](http://www.w3.org/TR/CSS2/media.html)(包括计算机屏幕和打印机）的**application.css**文件。对于一个有经验的Rails开发者，这行看起来很简单，但是起码有四个关于Ruby的用法让你感到迷惑：内建的Rails方法、没有圆括号、符号和hash。我们将在这章覆盖所有的这些概念。

另外，在视图中也包含其他大量的内建函数，而且Rails也允许创建新的函数。这样的函数称为*helper*。为了学习怎样开发一个自己的helper，让我们通过代码清单4.1里的代码来开始实验：

```ruby
<%= yield(:title) %> | Ruby on Rails Tutorial Sample App
```
上面的代码依赖于在每个视图里网页标题的定义（使用provide），和这里
```
<% provide(:title, "Home") %>
<h1>Sample App</h1>
<p>
  This is the home page for the
  <a href="http://www.railstutorial.org/">Ruby on Rails Tutorial</a>
  sample application.
</p>
```
但是假如我们不提供标题呢？好的编程实践是在每个页面有一个基础标题，假如我们想要更具体一点，可以再加一个可选的标题。我们几乎完成了之前的布局文件，不过有个小问题：如你所见，假如你删除了视图里的**provide**函数，缺少详细页面标题的完整标题就变成下面这样：
```
| Ruby on Rails Tutorial Sample App
```
换句话说，有一个合适的基础标题，但是在开头多了个字符“|”。

为了解决网页标题的问题，我们定义一个名为**full_title**的helper。假如没有定义页面标题，**full_title**helper返回基础标题，假如定义了的话，返回加了“|”的页面标题（代码清单4.2）。
```ruby
代码清单 4.2: 定义full_title helper.
# app/helpers/application_helper.rb
 module ApplicationHelper

  # Returns the full title on a per-page basis.
  def full_title(page_title = '')
    base_title = "Ruby on Rails Tutorial Sample App"
    if page_title.empty?
      base_title
    else
      page_title + " | " + base_title
    end
  end
end
```

现在我们定义了一个helper，我们可以用
```
<title><%= full_title(yield(:title)) %></title>
```
替换
<title><%= yield(:title) %> | Ruby on Rails Tutorial Samlple App</title>
如代码清单4.3所见。

```
代码清单 4.3: 使用full_title helper的网站布局文件。绿色
# app/views/layouts/application.html.erb
<!DOCTYPE html>
<html>
  <head>
    <title><%= full_title(yield(:title)) %></title>
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
为了让我们的helper正常工作，我们可以从主页消除不必要的单词“Home”，允许它使用基础标题。我们首先用代码清单4.4的代码更新一下我们的测试：更新了先前的标题测试和增加了一条标题里缺少“home”字符串的测试。
```
代码清单 4.4: 更新测试主页标题的测试。红色
# test/controllers/static_pages_controller_test.rb
require 'test_helper'

class StaticPagesControllerTest < ActionController::TestCase
  test "should get home" do
    get :home
    assert_response :success
    assert_select "title", "Ruby on Rails Tutorial Sample App"
  end

  test "should get help" do
    get :help
    assert_response :success
    assert_select "title", "Help | Ruby on Rails Tutorial Sample App"
  end

  test "should get about" do
    get :about
    assert_response :success
    assert_select "title", "About | Ruby on Rails Tutorial Sample App"
  end
end
```
让我们运行测试集来确认有一个测试失败：
```
代码清单 4.5: 红色
$ bundle exec rails test
3 tests, 6 assertions, 1 failures, 0 errors, 0 skips
```
为了让测试集通过，我们先从主页视图移除**provide**行，如代码清单4.6所见。
```
代码清单 4.6: The Home page with no custom page title. 绿色
# app/views/static_pages/home.html.erb
 <h1>Sample App</h1>
<p>
  This is the home page for the
  <a href="http://www.railstutorial.org/">Ruby on Rails Tutorial</a>
  sample application.
</p>
```
现在测试应该通过：
```
代码清单 4.7: 绿色
$ bundle exec rails test
```
(说明：先前的例子已经包含了运行**rails test**的部分输出，包括通过和失败测试的数量，但是为了简便起见，这些内容从这里开始就都忽略了。）

因为布局文件通过一行代码来引入了应用程序的样式表（CSS文件），代码清单4.2的代码可能对于有经验的Rails开发者来说很简单，但是它充满了重要的Ruby语法：模块、方法定义、可选的方法参数、注释、本地变量分配、布尔型、控制流、字符串连接以及返回值。这章也将覆盖所有这些内容。

## 4.2 字符串和方法
我们学习Ruby的主要工具是Rails控制台，如第一次在2.3.3节里看见的一样，用来和Rails应用程序交互的命令行工具。控制台是建立在交互Ruby（irb）基础之上的，因此可以使用Ruby语言的全部功能。（正如我们在4.4.4节中所见一样，控制台也可以进入Rails环境。）

假如你正使用云IDE，我建议你包含几个irb配置参数。使用简单的nano文本编辑器，在用户目录下创建.irbrc的文件，写入代码清单4.8里面的内容：
```
$ namo ~/.irbrc
```
代码清单4.8里的内容的作用是简化irb提示，压制一些烦人的自动缩进行为。
```
代码清单 4.8: Adding some irb configuration.
~/.irbrc
IRB.conf[:PROMPT_MODE] = :SIMPLE
IRB.conf[:AUTO_INDENT_MODE] = false
```

无论你是否包含了代码清单4.8里面的配置，你都可以在命令行启动控制台：
```
$ rails console
>>
```
默认情况下，控制台开启的是开发环境，那是Rails定义的三个独立的环境之一（其他环境是测试和生产环境）。这点在本章来说不重要，我们将在7.1.1节学到更多的环境。

控制台是一个伟大的学习工具，你可以自由地开发它的潜力--别担心，你（可能）不会破坏任何东西。当你使用控制台时，假如你被卡主了，按Ctrl-C退出程序、或者Ctrl-D退出控制台。在本章剩余部分，你可能会发现查看Ruby API是很有帮助的，它包含了丰富的信息（可能过于丰富了）。例如，为了学习更多的关于Ruby字符串的之后，你可以看看Ruby API中字符串类一节。

### 4.2.1 注释
Ruby注释前有个井号#（也叫“哈希标志”或者其他的），扩展到行末。Ruby忽略了注释，但是它对阅读代码者（也包括作者本人）有帮助。在代码里
```
# 返回在页面基本标题的基础上返回完整的标题
def full_title(page_title='')
.
.
.
end
```
第一行是注释，表明了后面的函数的功能。

你一般不用在使用控制台写注释，但是为了指导学习，我会在后面的代码里包含注释，像这样：
```
$ rails console
>> 17 + 42 #整数相加
=> 59
```

假如你跟随这节输入或者复制黏贴命令到自己的控制台，如果你想忽略注释的话，当然可以，反正控制台会忽略它们的。

### 4.2.2 字符串
字符串可能是网页应用程序里最重要的数据结构，因为网页基本上是由服务器发送至浏览的字符串组成。让我们开始在控制台探索字符串吧：

```ruby
$ rails console
>>"" #空字符串
=>""
>>"foo" #非空字符串
=>"foo"
```

这些是一串字符（有趣的是，叫字符串），使用双引号"创建（译者注：输入时注意，是英文双引号，不是中文双引号）。控制台输出每行的值，在这里一串字符正是字符串本身。

我们也可以使用操作符+来连接字符串：
```
>>"foo" + "bar" #字符串连接
=>"foobar"
```

这里输出的是"foo"加"bar"的结果字符串"foobar"。
另一个方法是通过插值使用特殊的语法“#{}”：
```
>> first_name="Michael" #变量赋值
=>“Michael”
>>"#{first_name} Hartl" #字符串插值
=> "Michael Hartl"
```

这里我们给变量first_name分配值“Michael”，然后把它插入字符串“#{first_name}
Hartl"。我们也可以把它们两个都分配个变量：

```
>> first_name = "Michael"
=> "Michael"
>> last_name = "Hartl"
=> "Hartl"
>> first_name + " " + last_name    # Concatenation, with a space in between
=> "Michael Hartl"
>> "#{first_name} #{last_name}"    # The equivalent interpolation
=> "Michael Hartl"
```
注意到两个表达式结果是相同的，但是我偏好插值版的；必须要加进一个空格好像有点尴尬。

#### 输出
最常用的输出字符串的Ruby函数是puts（发音：“put ess”，输出字符串）：
```
>> puts "foo"     # put string
foo
=> nil
```
puts方法有个副作用：表达式puts
"foo"把字符串输出到屏幕，然后返回空：nil是Ruby中特殊的值，表示什么也没有。（在后面的章节，为了简便，我有时会压制=>nil部分）

就像你在上个例子中看见的一样，使用puts自动第添加新行字符\n到输出。相关的print方法不会：
```
>> print "foo"    # 打印字符串 (和puts一样，但是结尾不换行）
foo=> nil
>> print "foo\n"  # 和puts "foo"一样
foo
=> nil
```
### 单引号字符串
到目前为止，所有的例子使用的是双引号括起来的字符串，但是Ruby也支持单引号括起来的字符串。在大部分时候，两种符号实际上是一样的：
```
>> 'foo'          # 单引号字符串
=> "foo"
>> 'foo' + 'bar'
=> "foobar"
```
不过有一个重要的不同，Ruby不会插入单引号字符串：
```
>> '#{foo} bar'     # 单引号字符串不允许插值
=> "\#{foo} bar"
```
注意控制台使用双引号返回值，但是在#{前使用转义字符反斜杠。

假如双引号字符串能做单引号字符串能做的任何事情、还可以进行插值运算，那么单引号字符串的意义是什么？因为它们是真正的字符，包含的恰好是你输入的字符串，这点常常很有用。例如，反斜杠字符在大多数系统都是特殊字符，如换行字符\n。假如你想用一个包含反斜杠字符的变量，单引号让这个变得很容易：
```
>> '\n'       # 文字版的'\n'
=> "\\n"
```

在我们先前的组合“#{}”的例子里，Ruby用一个额外的反斜杠转义反斜杠，在双引号字符串里，一个反斜杠字符要用两个反斜杠来表示。像这个小例子，没有省下多少力气，但是假如有许多反斜杠，那就真的有帮助了：
```
>> 'Newlines (\n) and tabs (\t) both use the backslash character \.'
=> "Newlines (\\n) and tabs (\\t) both use the backslash character \\."
```
最后，值得一提的是，在普通的情况下，单引号和双引号实际上是可以互换的，你将经常在源代码里看见两者的使用没有任何模式。关于这个真的没什么要做的了，除了说，“欢迎来到Ruby王国！”

### 4.2.3 对象和消息传输
在Ruby里，所有的一切都是对象。包括字符串、甚至nil，都是对象。我们将在4.4.2节里看到这个技术的意义，但是我认为任何人都不能通过看书里的定义就能理解，通过前面很多例子，我相信你已经有了一个直观的感受。

描述对象做什么很容易，它是对消息作出回应。像字符串这样的对象，例如，可以回应消息length，它会返回字符串的长度：

```
>> "foobar".length        # 把"length"当做消息传递给“foobar”
=> 6
```

典型地，传递到对象的消息叫方法，是定义在那些对象上的函数。字符串也有empty?方法：

```
>> "foobar".empty?
=> false
>> "".empty?
=> true
```
注意empty？方法末尾的问号。这是Ruby惯例表示返回值是逻辑型：true和false。逻辑型在控制流程方面尤其有用：

```
>> s = "foobar"
>> if s.empty?
>>   "The string is empty"
>> else
>>   "The string is nonempty"
>> end
=> "The string is nonempty"
```

为了包含几个语句，我们可以使用`elsif`（else + if）：

```
>> if s.nil?
>>   "The variable is nil"
>> elsif s.empty?
>>   "The string is empty"
>> elsif s.include?("foo")
>>   "The string includes 'foo'"
>> end
=> "The string includes 'foo'"
```
逻辑型也可以通过使用&&（和）、||（或）和！（非）操作符：
```
>> x = "foo"
=> "foo"
>> y = ""
=> ""
>> puts "Both strings are empty" if x.empty? && y.empty?
=> nil
>> puts "One of the strings is empty" if x.empty? || y.empty?
"One of the strings is empty"
=> nil
>> puts "x is not empty" if !x.empty?
"x is not empty"
=> nil
```
因为在Ruby中所有的东西都是对象，所以nil也是对象，所以它也有方法。另外一个例子是to_s方法能将任何对象都转换成字符串：
```
>> nil.to_s
=> ""
```
这肯定显示是一个空的字符串，我们可以通过方法链接来传递给nil的消息来确认：
```
>> nil.empty?
NoMethodError: undefined method `empty?' for nil:NilClass
>> nil.to_s.empty?      # 方法串联
=> true
```

我们在这看见nil对象对empty？方法没有响应，但是nil.to_s响应了。
有一个特殊的测试值是nil的方法，你可能猜到了：
```
>> "foo".nil?
=> false
>> "".nil?
=> false
>> nil.nil?
=> true
```
代码
```
puts "x is not empty" if x.empty?
```
也显示了使用if关键词的变化：Ruby允许你这样写：仅仅当声明后面的if语句是true时才执行前面的代码。有个互补的关键词unless，工作方法一样：
```
>> string = "foobar"
>> puts "The string '#{string}' is nonempty." unless string.empty?
The string 'foobar' is nonempty.
=> nil
```
nil对象是特殊的，因为它是Ruby中所有对象里除了false之外，唯一值为false的对象。我们可以看见这个用法**!!**(读“bang bang”），这个否定一个对象两次，强迫一个变量转换为逻辑型值：
```
>> !!nil
=> false

也就是说，所有别的Ruby对象都是true，甚至0也是。

### 4.2.4 定义方法

控制台允许我们和代码清单3.6的**home**动作或代码清单4.2中**full_title**辅助方法一样定义方法。(在控制台里定义方法有点啰嗦，通常你会用文件，但是为了方便说明，我们这里用控制台）例如，让我们定义函数**string_message**，这个函数有一个参数，最后根据是否参数是空返回信息：
```ruby
>> def string_message(str = '')
>>   if str.empty?
>>     "It's an empty string!"
>>   else
>>     "The string is nonempty."
>>   end
>> end
=> :string_message
>> puts string_message("foobar")
The string is nonempty.
>> puts string_message("")
It's an empty string!
>> puts string_message
It's an empty string!
```

如在最后的例子里所见，也可以不传递任何参数（这时我们可以忽略括号）。这是因为代码：
```ruby
def string_message(str = '')
```
包含一个默认的参数，空字符串。这使得**str**参数可选，假如我们没有传输参数，就会使用默认值。

注意Ruby函数有隐含的返回，也就是说它会返回最后一条语句的值--在这里是依据方法的参数**str**是空还是非空，返回两个字符串之一。Ruby也可以用明示的方式返回；下面的函数和上面的是一样的：
```ruby
>> def string_message(str = '')
>>   return "It's an empty string!" if str.empty?
>>   return "The string is nonempty."
>> end
```
(聪明的读者可能已经注意到第二个**return**实际上是不必要的--作为函数的表达式，字符串“The
string is nonempty.”不管有没有关键词**return**都会返回值，但是在两个地方都使用**return**看起来有种对称美。）

理解函数参数的名称是和调用者关心的不相干的也是重要的。换句话说，上面的第一个例子能用任何有效的变量名替换**str**，例如**the_function_argument**，和之前的行为是一模一样的。
```ruby
>> def string_message(the_function_argument = '')
>>   if the_function_argument.empty?
>>     "It's an empty string!"
>>   else
>>     "The string is nonempty."
>>   end
>> end
=> nil
>> puts string_message("")
It's an empty string!
>> puts string_message("foobar")
The string is nonempty.
```

### 4.2.5 回到标题辅助函数
我们现在到了理解代码清单4.2中**full_title**辅助函数的时候了，代码清单4.9中用注释说明了每行代码的意义。
```ruby
代码清单 4.9: 注释好的title_helper。
# app/helpers/application_helper.rb
 module ApplicationHelper

  # 返回基于每页基本标题的全标题                      # 文档注释
  def full_title(page_title = '')                     # 方法 def，可选参数
    base_title = "Ruby on Rails Tutorial Sample App"  # 变量分配
    if page_title.empty?                              # 逻辑测试
      base_title                                      # 隐含返回
    else
      page_title + " | " + base_title                 # 字符串连接
    end
  end
end
```

这些要素--函数定义（带可选参数）、变量分配、逻辑测试、控制流和字符串连接--一起为我们网站的布局文件组成了紧凑的辅助方法。最后的要素是**module ApplicationHelper**：模块（module）是让我们把相关的方法打包在一起的一种方法，这些方法然后可以使用**include**混合进Ruby类中。当写Ruby程序时，你常常会先写模块，然后显示地把它们包含进来，但是在这个例子中，辅助方法模块Rails为我们处理了包含动作，所以辅助方法full_title**在我们的所有视图里自动可用了。

## 4.3 其他数据结构
尽管网页应用归根到底是字符串，实际上创建那些字符串也需要别的数据结构。在这一节，我们将学习一些对Rails应用程序来说相对重要的Ruby数据结构。

### 4.3.1 数组和范围
数组就是按照一定顺序排列的列表。我们还没有在本书中讨论数组，但是理解它们会为理解哈希（4.3.3节）奠定良好的基础，为Rails的数据模块化（例如在2.3.3节里看见的**has_many**关联和在11.1.3节里将会有更多讨论）。

到目前为止我们已经花了许多时间理解字符串，有一个名为**split**的方法可以自然地把字符串转化为数组：
```ruby
>> "foo bar    baz".split #把字符串分成包含三个元素的数组
=> ["foo", "bar", "baz"]
```
这个操作的结果是包含三个字符串的数组。默认地，**split**使用空格作为分隔符，把字符串分成数组，但是你也可以用几乎任何字符作为分隔符：
```ruby
>> "fooxbarxbazx".split('x')
=> ["foo", "bar", "baz"]
```
和许多其他计算机语言遵循的惯例一样，Ruby数组从0开始编号，这意味着在数组里第一个元素的索引是0，第二个是1，以此类推：
```ruby
>> a = [42, 8, 17]
=> [42, 8, 17]
>> a[0]               # Ruby使用方括号取数组元素
=> 42
>> a[1]
=> 8
>> a[2]
=> 17
>> a[-1]              # 索引甚至可以是负的！
=> 17
```

我们看见Ruby使用方括号读取数组元素。另外，Ruby也为方括号提供了对称的方法取元素：
```
>> a                  # 回忆一下'a' 是什么
=> [42, 8, 17]
>> a.first
=> 42
>> a.second
=> 8
>> a.last
=> 17
>> a.last == a[-1]    # 使用==比较
=> true
```

最后一行介绍了等于比较符号*==*，Ruby和其他语言一样，沿用**！=**表示不相等，等等：
```
>> x = a.length
=> 3
>> x == 3
=> true
>> x ==1
=> false
>> x != 1
=> true
>> x >= 1
=> true
>> x < 1
=> false
```
除了**length**(上面第一行），数组还有很多其他方法：
```ruby
>> a
=> [42, 8, 17]
>> a.empty?
=> false
>> a.include?(42)
=> true
>> a.sort
[8, 17, 42]
>> a.reverse
=> [17, 8 . 42]
>> a.shuffle
=> [17, 42, 8] #这个结果是随机，如果与此不同请不要担心
a
=> [42, 8, 17]
```

注意：上面的方法都没有改变a本身。为了改变数组，要使用带感叹号的方法（在这里，感叹号的发音是“bang”）：
```ruby
>> a
=> [42, 8, 17]
>> a.sort!
=> [8, 17, 42]
>> a
=> [8, 17, 42]
```
你也可以使用**push**方法或效果相同的操作符， <<:
```ruby
>> a.push(6)                  # 把6放进数组
=> [42, 8, 17, 6]
>> a << 7                     # 把7放进数组
=> [42, 8, 17, 6, 7]
>> a << "foo" << "bar"        # 链式压进数组
=> [42, 8, 17, 6, 7, "foo", "bar"]
```

最后例子显示你可以串联一起，将元素压进数组，而且也不像别的语言里的数组，Ruby中得数组可以包含不同类型的元素（在这个例子里，整数和字符串）。
之前我们看到**split**将字符串转为数组。我们也可以使用**join**方法把数组变成字符串：
```
>> a
=> [42, 8, 17, 7, "foo", "bar"]
>> a.join              #什么也不用合并
=> "428177foobar"
>> a.join(',')         #用逗号合并
=> "42, 8, 17, 7, foo, bar"
```

和数组相关的另一个种数据结构是范围（rang），通过使用**to_a**方法把范围转换为数组可能最容易让人理解：
```ruby
>> 0..9
=> 0..9
>> 0..9.to_a    #糟糕，在9上使用to_a函数
NoMethodError: undefined method `to_a' for 9:Fixnum   #没有定义此方法
>> (0..9).to_a  # 使用圆括号在范围上调用to_a
=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```
尽管0..9是有效的范围，上面第二个表达式显示我们在调用方法的时候需要使用圆括号。

范围对于去除数组元素很有用：
```
>> a = %w[foo bar baz quux]    #使用%w来创建一个字符串数组
=> ["foo", "bar", "baz", "quux"]
>> a[0..2]
=> ["foo", "bar", "baz"]
```

一个尤其有用的技巧是在范围的末尾使用索引-1来选择从开始到结尾的每个要素，不需要显示第使用数组长度：
```
>> a = (0..9).to_a
=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>> a[2..(a.length-1)]               # 显示地使用数组长度
=> [2, 3, 4, 5, 6, 7, 8, 9]
>> a[2..-1]                         # 使用技巧索引-1
=> [2, 3, 4, 5, 6, 7, 8, 9]
```
范围对字符也有效：
```
>> ('a'..'e').to_a
=> ["a", "b", "c", "d", "e"]
```

### 4.3.2 块（block）
数组和范围有许多方法接受块作为参数，块几乎是Ruby最强大，最有迷惑性的特性：
```ruby
>> (1..5).each { |i| puts 2 * i }
2
4
6
8
10
=> 1..5
```
这行代码在范围**(1..5)**上调用**each**方法，然后把它传递给块**{ |i| puts 2 * i }。在**|i|**里，变量名称两边的竖线是Ruby中块变量的语法，决定了块对方法做什么。
在这个例子里，范围的**each**方法可以使用一个局部变量处理块，我们定义为**i**，它为范围里的每个值执行块。

大括号是声明块的方法之一，还有另外一种方法：
>> (1..5).each do |i|
?> puts 2 * i
>> end
2
4
6
8
10
=> 1..5
```
块中的代码可以多余一行，事实上常常多于一行。在本书中，我们将遵循惯例，如果块仅有一行代码，就使用大括号{}，否则就用**do..end**语法：
```
>> (1..5).each do |number|
?>   puts 2 * number
>>   puts '--'
>> end
2
--
4
--
6
--
8
--
10
--
=> 1..5
```
这里我使用**number**替换**i**是为了强调我们可以用任何变量名。

除非你有很深的编程方面的背景，否则理解块是没有捷径的；你不得不看许多它们的例子，最终你将学会使用它们。幸运的是，人们擅长归纳；这里有几个使用**map**方法的例子：
```ruby
>> 3.times { puts "Betelgeuse!" }   # 3.times以一个没有变量的块作为参数。
"Betelgeuse!"
"Betelgeuse!"
"Betelgeuse!"
=> 3
>> (1..5).map { |i| i**2 }          # The ** notation is for 'power'.
=> [1, 4, 9, 16, 25]
>> %w[a b c]                        # Recall that %w makes string arrays.
=> ["a", "b", "c"]
>> %w[a b c].map { |char| char.upcase }
=> ["A", "B", "C"]
>> %w[A B C].map { |char| char.downcase }
=> ["a", "b", "c"]
```
正如你所见，**map**方法返回数组或范围里的每个元素执行块中的代码后的值。最后两个例子里，**map**里的块对块变量调用一个特别的方法，在这里有一个常用的简写：
```
>> %w[A B C].map { |char| char.downcase }
=> ["a", "b", "c"]
>> %w[A B C].map(&:downcase)
=> ["a", "b", "c"]
```
(这个看上去奇怪，压缩的代码使用了符号（symbol），我们将在4.3.3节中讨论）关于这个结构有趣的一件事情是最初是Ruby
on Rails加进来的，人们太喜欢它了，所以后来加进了Ruby内核中。

作为我们最后一个块例子，我们看一下代码清单4.4中的测试：
```
test "should get home" do
  get :home
  assert_response :success
  assert_select "title", "Ruby on Rails Tutorial Sample App"
end
```

理解细节不重要（实际上我不知道细节），但是我们可以从关键词**do**来推断测试的主体是块。**test**方法带一个字符串作为参数（描述）和一个块，然后执行块的主体，作为测试集的一部分。

顺便提一下，我们现在应该可以理解1.5.4节中生产随机子域名的方法了：
```
('a'..'z').to_a.shuffle[0..7].join
```
让我们一步步来：
```
>> ('a'..'z').to_a                     # 字母表数组
=> ["a", "b", "c", "d", "e", "f", "g", "h", "i", "j", "k", "l", "m", "n", "o",
"p", "q", "r", "s", "t", "u", "v", "w", "x", "y", "z"]
>> ('a'..'z').to_a.shuffle             # 弄乱它
=> ["c", "g", "l", "k", "h", "z", "s", "i", "n", "d", "y", "u", "t", "j", "q",
"b", "r", "o", "f", "e", "w", "v", "m", "a", "x", "p"]
>> ('a'..'z').to_a.shuffle[0..7]       # 取出前8个元素。
=> ["f", "w", "i", "a", "h", "p", "c", "x"]
>> ('a'..'z').to_a.shuffle[0..7].join  # 把它们组合起来组成字符串
=> "mznpybuj"
```
    
    注：shuffle方法返回的值是随机的，你最后得到的字符串可能和这里不一样。

### 4.3.3 哈希表和符号
哈希表本质上是数组。哈希的键，也叫做索引，几乎可以是任何对象。例如，我们可以使用字符串当作键：

```ruby
>> user = {}                         # {}是空的哈希
=> {}
>> user["first_name"] = "Michael"    # 键“first_name”， 值“Michael”
=> "Michael"
>> user["last_name"] = "Hartl“       # 键“last_name”,   值“Hartl”
=> "Hartl"
>> user["first_name"]                 # Element access is like arrays.
=> "Michael"
>> user                               # A literal representation of the hash
=> {"last_name"=>"Hartl", "first_name"=>"Michael"}
```
哈希用大括号声明，包含键-值对；没有键值的一对大括号表示空哈希。注意哈希使用的大括号和块使用的大括号没有任何关系。
(是的，这有点让人困惑。）尽管和数组类似，一个重要的不同是哈希一般不能保证它的元素会保持一定的顺序。假如要用到顺序，就只能使用数组了。
通过方括号的方式可以每次定义一条Hash对，不过这种方式不太方便。使用=>定义Hash键值要方便的多，**=>**成为“哈希火箭（hashrocket）”：
```ruby
>> user = { "first_name" => "Michael", "last_name" => "Hartl" }
=> {"last_name"=>"Hartl", "first_name"=>"Michael"}
```

这里我使用Ruby的通常做法：在哈希的两端添加额外的空格--会被控制台输出忽略的惯例。（不要问我为什么空格是惯例，可能一些早期有影响的Ruby程序员喜欢有额外空格的表示，然后就形成了惯例。）

到目前，我已经使用字符串作为哈希键值，但是在Rails里，更平常的是使用符号。符号看起来像是字符串，但是前面有个冒号，而不是引号。例如，:name是符号。你可以认为符号是基础的字符串，没有其他多余的包袱（方法）：
```ruby
>> "name".split('')
=> ["n", "a", "m", "e"]
>> :name.split('')
NoMethodError: undefined method `split' for :name:Symbol
>> "foobar".reverse
=> "raboof"
>> :foobar.reverse
NoMethodError: undefined method `reverse' for :foobar:Symbol
```
符号是特殊的Ruby数据类型，很少可以和其他语言共享。所以刚开始可能觉得奇怪，但是Rails经常使用它们，所以你会很快学会使用它们。不像字符串，不是所有的字符都是有效的符号：
```ruby
>> :foo-bar
NameError: undefined local variable or method `bar' for main:Object
>> :2foo
SyntaxError
```
只要你用字母开始，然后使用普通的字符，你应该没事。

关于符号作为哈希键值，我们可以定义一个**user**哈希如下：
```ruby
>> user = { :name => "Michael Hartl", :email => "michael@example.com" }
=> {:name=>"Michael Hartl", :email=>"michael@example.com"}
>> user[:name]              # 进入键:name对应的值
=> "Michael Hartl"
>> user[:password]          # 进入未定义的键返回nil
=> nil
```
我们从最后的例子看见，如果没有定义键，它的值是nil。

因为使用符号作为键值非常普遍，在版本1.9的Ruby为这种特例，支持一个新的语法：
```ruby
>> h1 = { :name => "Michael Hartl", :email => "michael@example.com" }
=> {:name=>"Michael Hartl", :email=>"michael@example.com"}
>> h2 = { name: "Michael Hartl", email: "michael@example.com" }
=> {:name=>"Michael Hartl", :email=>"michael@example.com"}
>> h1 == h2
=> true
```
第二种语法用键名后面加一个冒号和值替换符号/哈希火箭组合的表示方法：
```ruby
{ name： "Michael Hartl", email: "michael@example.com" }
```

这种方法更接近其他语言的哈希声明（例如Javascript），欣赏这种方式的程序员在Rails社区里越来越多。因为两种哈希语法在使用上仍然很普遍，所以能认出这两种语法很有必要。
不幸的是，这可能有点让人困惑，尤其因为**:name**是有效地符号，但是**name:**本身没有任何意义。底线是**:name =>**和**name:**效率是一样的，因此
```ruby
{ :name => "Michael Hartl" }
```
和
```ruby
{ name: "Michael Hartl" }
```
是相同的，否则你需要使用**:name**（用冒号开头）来表明它是符号。
哈希的值可以是任何对象，甚至是其它哈希也可以，如同在代码清单4.10所示。
```
代码清单 4.10: 嵌套Hash
>> params = {}        # 定义一个名为'params'( 'parameters'的简写)的哈希。
=> {}
>> params[:user] = { name: "Michael Hartl", email: "mhartl@example.com" }
=> {:name=>"Michael Hartl", :email=>"mhartl@example.com"}
>> params
=> {:user=>{:name=>"Michael Hartl", :email=>"mhartl@example.com"}}
>>  params[:user][:email]
=> "mhartl@example.com"
```

这种哈希的哈希，或者叫嵌套哈希，在Rails中大量第使用，我们将在7.3节开始看见。

和数组和范围一样，哈希一样有**each**方法。例如，考虑一个名叫**flash**的哈希，有两组元素，**:success**和**:danger**：
```ruby
>> flash = { success: "It worked!", danger: "It failed." }
=> {:success=>"It worked!", :danger=>"It failed."}
>> flash.each do |key, value|
?>   puts "Key #{key.inspect} has value #{value.inspect}"
>> end
Key :success has value "It worked!"
Key :danger has value "It failed."
```
记住，数组的each方法是仅带一个变量的块，哈希的each方法有两个参数，键和值。这样，哈希的each方法遍历哈希一次一组键-值对。

最后的例子使用非常有用的**inspect**方法，它会返回字符串表示的对象：
```
>> puts (1..5).to_a            # 把数组当做字符串输出
1
2
3
4
5
>> puts (1..5).to_a.inspect    # 按我们输入的方式输出数组
[1, 2, 3, 4, 5]
>> puts :name, :name.inspect
name
:name
>> puts "It worked!", "It worked!".inspect
It worked!
"It worked!"
```
顺便提一下，使用**puts**来输出对象非常普通，以至于它有一个简写，函数p：
```
>> p :name
:name
```

### 4.3.4 重访CSS
现在是时候重访代码清单4.1，在布局文件中包含层叠样式表（CSS）:
```
<%= stylesheet_link_tag 'application', media: 'all',
                                       'data-turbolinks-track' => true %>
```
我们现在几乎可以理解这句了。如同在4.1节中简短地提到的一样，Rails定义了一个特殊函数来包含样式，如
```ruby
stylesheet_link_tag 'application', media: 'all',
                                   'data-turbolinks-track' => true
```
是调用这个函数。但是有几个迷题：首先，圆括号到哪里了？在Ruby中，它们是可选的，所以这两个是等价的：
```ruby
# 函数调用时圆括号是可选的
stylesheet_link_tag('application', media: 'all',
                                   'data-turbolinks-track' => true)
stylesheet_link_tag 'application', media: 'all',
                                   'data-turbolinks-track' => true
```
其次，**media**参数看起来很像哈希，但是大括号去那里了？当哈希是函数最后的参数，大括号是可选的，所以它们两个是等价的：
```ruby
# 如果函数最后的参数是哈希时，大括号是可选的
stylesheet_link_tag 'application', { media: 'all',
                                     'data-turbolinks-track' => true }
stylesheet_link_tag 'application', media: 'all',
                                   'data-turbolinks-track' => true
```
接下来，为什么**date-turbolinks-track**使用旧式的哈希火箭语法？这是因为使用新语法写
```ruby
data-turbolinks-track: true
```
是无效的，因为连字符的关系，data-turbolinks-track不是有效的符号。（在4.3.3节提到过连字符不能用在符号中。）这迫使我们使用旧式语法，变成了
```ruby
'data-turbolinks-track' => true
```
最后，为什么Ruby正确地解释这行
```ruby
stylesheet_link_tag 'application', media: 'all',
                                   'data-turbolinks-track' => true
```
甚至在最后一行断开？答案是Ruby在这种环境下不区分换行符和别的空格。我选择把代码分成两行的原因是为了清晰。
我偏好把每行源代码的长度控制在80个字符内。

所以，我们继续往下看见这行
```
stylesheet_link_tag 'application', media: 'all',
                                   'data-turbolinks-track' => true
```
调用**stylesheet_link_tag**函数，带两个参数：字符串，表明样式表的路径，一个有两个元素的哈希表，表明媒体类型和告诉Rails使用**turbolinks**特性，它是在Rails4.0加进来的。
因为<%= %>括号，结果被插入ERb模板，假如你在浏览器里查看本页的源代码，你会看见HTML需要包含一个样式表（代码清单4.11）。（你肯定也看见其他的内容，如**?body=1**，跟在CSS文件名后面。这些是Rails插入的，用来确保当文件改变之后浏览器可以重新加载CSS文件。）
```
代码清单 4.11: 引入CSS函数输出的HTML代码。
<link data-turbolinks-track="true" href="/assets/application.css" media="all"
rel="stylesheet" />
```
假如你真的查看[http://localhost:3000/assets/application.css](http://localhost:3000/assets/application.css)这个文件,你会看见（除了注释）它是空的。我们将在第五章改变它。

## 4.4 Ruby类
我们之前说过在Ruby中所有的东西都是对象，在这节我们将在最后定义一些我们自己的类。Ruby，像其他许多面向对象的语言，使用类来组织方法；这些类然后被初始化，创建对象。假如你刚接触面向对象的编程语言，这听起来可能像胡扯，所以我们看一些具体的例子吧。

### 4.4.1 构造函数
我们已经看见过许多使用类初始化对象的例子了，但是我们仍然显示地实现。
例如，我们使用双引号初始化一个字符串，它就是字符串的构造函数：
```
>> s = "foobar"       # 双引号对于字符串来说是真的构造函。
=> "foobar"
>> s.class
=> String
```
我们这里看见字符串响应方法**class**，只是返回它属于哪个类。

与使用字面的构造函数不同，我们使用等价的命名的构造函数，它在类名上调用**new**方法：
```
>> s = String.new("foobar")   #为字符串命名的构造函数。
=> "foobar"
>> s.class
=> String
>> s == "foobar"
=> true
```

这和字面构造函数是等价的，但是关于我们做得事更明显。
数组和字符串工作方法一样：
```
>> a = Array.new([1, 3, 2])
=> [1, 3, 2]
```
哈希，相反地，是不同的。当数组构造函数**Array.new**为数组传递一个初始值，**Hash.new**为哈希传入一个默认的值，这个值是哈希为不存在的键的默认值：
```
>> h = Hash.new
=> {}
>> h[:foo]
=> nil
>> h = Hash.new(0)
=> {}
>> h[:foo]
=> 0
```

当类直接调用方法时，如这里的**new**，它在调用类方法。在类上调用**new**是类的对象，也叫类的实例。在实例上调用的方法，例如**length**，叫实例方法。

### 4.4.2 类继承
当学习使用类，使用**superclass**方法查明类的继承是很有用的：
```
>> s = String.new("foobar")
=> "foobar"
>> s.class                        # Find the class of s.
=> String
>> s.class.superclass             # Find the superclass of String.
=> Object
>> s.class.superclass.superclass  # Ruby 1.9 uses a new BasicObject base class
=> BasicObject
>> s.class.superclass.superclass.superclass
=> nil
```
继承关系如图4.1显示。我们这里看见**String**的超级类是**Object**，**Object**的是**BasicObject**，但是**BasicObject**没有超类。这个模式对每个Ruby对象来说都是真的：回溯类继承关系足够远，每个Ruby类最终都继承于**BasicObject**，它本身没有超类。这是“Ruby中所有的都是对象”的技术意思。

![图4.1：String类的继承等级](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/string_inheritance_ruby_1_9.png)

为了更深地理解类，创建我们自己的类是不二的选择。让我们创建一个**Word**类，类里有一个名为**palindrome?**的方法，该方法返回**true**，假如单词从前和从后写都是一样的话：
```
>> class Word
>>   def palindrome?(string)
>>     string == string.reverse
>>   end
>> end
=> :palindrome?

```
我们可以如下使用它：
```
>> w = Word.new              # Make a new Word object.
=> #<Word:0x22d0b20>
>> w.palindrome?("foobar")
=> false
>> w.palindrome?("level")
=> true
```
假如这个例子让你觉得有点牵强，挺好--这是故意设计的。创建一个只有一个带字符串为参数方法的类是有点奇怪。因为单词是字符串，**Word**继承于**String**更自然，如代码清单4.12所示。（你应该退出控制台然后重新进入，这样可以清除旧的**Word**定义。）

```
代码清单 4.12: 在控制台定义Word类。
>> class Word < String             # Word继承与字符串String。
>>   # 假如字符串是回文字符串则返回true
>>   def palindrome?
>>     self == self.reverse        # self is the string itself.
>>   end
>> end
=> nil
```
这里**Word < String**是Ruby继承类的语法（在3.2节中简断地讨论过），它确保了除了新的**palindrome？**方法，单词也有字符串一样的方法：
```
>> s = Word.new("level")    # Make a new Word, initialized with "level".
=> "level"
>> s.palindrome?            # Words have the palindrome? method.
=> true
>> s.length                 # Words also inherit all the normal string methods.
=> 5
```

因为**Word**类继承了**String**类，我们用控制台显示地看看类的继承等级：
```
>> s.class
=> Word
>> s.class.superclass
=> String
>> s.class.superclass.superclass
=> Object
```

图4.2阐明了继承。
![图4.2：对于代码清单4.12里的Word类（不是内建的）继承等级](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/word_inheritance_ruby_1_9.png)
Rubyye 允许我们使用**self**关键词：在**Word**类里，**self**是对象自己，这意味着我们可以用
```ruby
self == self.reverse
```

来检查是否单词是回文。实际上，在字符串类中在方法和属性上使用**self.**是可选的，（除非我们创建一个赋值），所以
```ruby
self = reverse
```
一样工作。

### 修改内建类

虽然继承是一个有力的方式，在回文的例子中，可能把方法直接加入**String**类可能更自然，以便（在其他情况）我们可以在字符串上调用**palindrome?**。现在我们做不到这个：
```
>> "level".palindrome?
NoMethodError: undefined method `palindrome?' for "level":String
```

令人惊奇的是，Ruby允许你这样做；Ruby类是开放类，可以修改，允许普通人，例如我们给它们添加方法：
```ruby
>> class String
>>   # Returns true if the string is its own reverse.
>>   def palindrome?
>>     self == self.reverse
>>   end
>> end
=> nil
>> "deified".palindrome?
=> true
```
(我不知道那个更酷：是Ruby让你添加方法到内建类，还是**deified**是回文。）

修改内建类是强大的技术，但是能力越大，责任越大，如果没有非常好的理由，那么给内建类添加方法是非常不推荐的。Rails确实有一些好的理由；例如，在Web应用程序里，我们常常想要阻止变量是空得--例如，用户名不能是空的--所以Rails加了一个**blank?**方法到Ruby。因为Rails控制台自动包含了Rails扩展，我们可以看看这里的例子（这在irb下不工作）：
```ruby
>> "".blank?
=> true
>> "   ".empty?
=>false
>> "   ".blank?
=>true
>> nil.blank?
=> true
```

我们看见空格字符串不是空的，但是它是空白的。注意**nil**是空的；因为**nil**不是字符串，这暗示Rails实际上把**blank?**到**String**的基类，就是（就像我们在这节开始看到的一样）是**Object**。我们将在8.4节看到Rails其他给Ruby内建类添加方法的例子。

### 4.4.4 控制器类
所有这些关于类和继承可能已经激发了认知的闪光，因为之前我们已经看见两者了，在类StaticPagesController中（代码清单3.18）：
``` Ruby
class StaticPagesController < ApplicationController

  def home
  end

  def help
  end

  def about
  end
end
```
你现在可以理解它了，起码模糊地，代码的意义：**StaticPagesController**是一个继承于**ApplicationController**类，它有**home，help，和about**几个方法。
因为每个Rails控制台的session加载了本地的Rails环境，我们甚至可以显示地创建一个控制器，然后检查它的继承：
```
>> controller = StaticPagesController.new
=> #<StaticPagesController:0x007f82cffb7f58 @_action_has_layout=true,
@_routes=nil, @_headers={"Content-Type"=>"text/html"}, @_status=200,
@_request=nil, @_response=nil>
>> controller.class
=> StaticPagesController
>> controller.class.superclass
=> ApplicationController
>> controller.class.superclass.superclass
=> ActionController::Base
>> controller.class.superclass.superclass.superclass
=> ActionController::Metal
>> controller.class.superclass.superclass.superclass.superclass
=> AbstractController::Base
>> controller.class.superclass.superclass.superclass.superclass.superclass
=> Object
```
继承结构如图4.3所示。
![图4.3：静态页面的继承等级](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/static_pages_controller_inheritance.png)
我们甚至可以在控制台里调用控制器动作，它们只是方法：
```
>> controller.home
=> nil
```

这里因为**home**动作是空的所以返回**nil**。

但是等等--动作没有返回值，起码没有返回值得关心的值。**home**动作，正如我们在第三章看见的一样，渲染了一个网页，而不是返回一个值。而且我确定不记得曾经在任何地方调用过**StaticPagesController.new**。发生什么了？

发生了什么是Rails是用Ruby写的，但是Rails不是Ruby。有些Rails类像普通的Ruby对象，但是有些只是Rails魔法山的材料。Rails是自成一格的，应该独立于Ruby学习和理解。


### 4.4.5 User类
我们通过编写一个完整的类来结束我们的Ruby之旅，**User**类和第六章的User模型的先驱。

到目前，我们都是在控制台进行类的定义，但是你应该很快就感觉很烦了。所以，我们现在直接在应用程序的根目录下创建文件**example_user.rb**，然后写入代码清单4.13里面的代码：
```
代码清单 4.13: User类的代码
# example_user.rb
 class User
  attr_accessor :name, :email

  def initialize(attributes = {})
    @name  = attributes[:name]
    @email = attributes[:email]
  end

  def formatted_email
    "#{@name} <#{@email}>"
  end
end
```
这里有点长，所以我们一步步来。第一行，
```ruby
attr_accessor :name, :email
```
声明了可以读取属性:name和:email。这条语句创建了“getter”和“setter”方法，允许我们读取和设置@name和@email实例变量，我们在2.2.2节和3.6节中简单地提到过。
在Rails里，实例变量最主要的特性是它们自动就可以在视图中使用了。但是通常来说，它们是在Ruby类中可随意调用的变量。（关于这个还有更多知识需要讲。）实例变量总是以**@**开始，当定义时默认的值为**nil**。

接下来我们再看看类里面的第一个方法，**initialize**。它在Ruby中是特殊的一个方法：它是我们在执行**User.new**时执行的第一个方法。这个特殊的**initialize**方法有一个参数，**arrtributes**：

```ruby
def initialize(attributes = {})
  @name  = attributes[:name]
  @email = attributes[:email]
end
```
这里**attrbiuttes**变量是默认值为空的哈希，以便我们能在没有传递name和email的情况下定义一个用户。（回忆4.3.3节，哈希为不存在的键返回**nil**，所以假如没有设置**attributes[:name]**的话，**attributes[:name]**值将默认为时**nil**，**attributes[:email]**也类似。）

最后，我们的类定义了一个名为**formatted_email**的方法，使用字符串插值的方法用@name和@email（4.2.2节）建立了一个格式化好得用户名、email地址的版本：

```
def formatted_email
  "#{@name} < #{@email}>"
end
```
因为@name和@email两个都是实例变量（用@标记表明的），它们在**formatted_email**方法里是自动可用的。

让我们发动控制台，**require**用户例子的代码，然后把我们的用户类取出来溜溜：
```ruby
>> require './example_user'
=> true
>> example = User.new
>> #<User:0x224ceec @email=nil, @name=nil>
>> example.name
=> nil
>> example.name = "Example User"
=> "Example User"
>> example.email = "user@example.com"
=> "user@example.com"
>> example.formatted_email
=> "Example User < user@example.com>"
```

这里**‘.’**是Unix表示“当前目录”，**'/example_user'**告诉Ruby在相对位置查找例子用户文件。随后的代码创建了一个空得用户例子，然后填入名字和email地址通过直接给相应的属性赋值（通过代码清单4.13里的**attr_accessor**，赋值变得可能）。当我们写
```ruby
example.name = "Example User"
```
Ruby正给变量**@name**赋值**"Example User"**(**email**属性也是一样的），我们然后会在**formatted_email**方法里使用。

回忆4.3.4节，我们可以忽略最后的哈希参数的大括号，然后我们通过直接给**initialize**方法传递哈希参数创建一个预先定义属性的用户：
```ruby
>> user = User.new(name: "Michael Hartl", email: "mhartl@example.com")
=> #<User:0x225167c @email="mhartl@example.com", @name="Michael Hartl">
>> user.formatted_email
=> "Michael Hartl <mhartl@example.com>"
```

我们在第七章会看到使用哈希参数初始化对象--在普通的Rails应用程序里常见的技术--就做集中赋值。


## 4.5 结语
这里结束我们Ruby语言的概览。在第五章，我们准备开始在开发示例程序时好好使用这些技能。

我们不会再使用4.4.5节用过的**example_user.rb**文件了，所以我建议删除它：
```
$ rm example_user.rb
```

然后把其他改变提交到主要的源代码仓库，推送到Bitbucket, 然后部署到Heroku：
```
$ git status
$ git commit -am "Add a full_title helper"
$ git push
$ bundle exec rails test
$ git push heroku
```

### 4.5.1 我们在这章学到了什么
* Ruby有许多操作字符串的方法
* 在Ruby里所有的都是对象
* Ruby使用**def**定义方法
* Ruby使用**class**定义类
* Rails视图可以包含静态HTML和内嵌Ruby（ERb）
* 内建的Ruby数据结构包含数组，范围和哈希
* Ruby块是易用的结构，允许自然遍历可遍历的数据结构
* 符号式标签，像没有额外结构的字符串
* Ruby支持对象继承
* 打开和修改Ruby内建类是可能的
* 单词“deified”是回文

## 4.6 练习
1. 通过用合适的方法替换代码清单4.14里的问号，联合**split，shuffle和join**写一个生成给定字符串的随机字符。
2. 使用代码清单4.15为向导，把**shuffle**方法添加到**String**类。
3. 创建三个哈希：person1, person2, person3。用键:first, :last表示名字和姓。然后创建一个参数哈希，以便params[:father]是person1,
params[:mother]是person2，params[:child]是person3。确认，例如，params[:fater][:first]有正确的值。
4. 查看在线版的Ruby API，阅读关于哈希方法**merge**。下面表达式的值是什么？
```ruby
{ "a" => 100, "b" => 200}.merge({ "b" => 300 })
```

```ruby
代码清单 4.14: string shuffle函数的框架
>> def string_shuffle(s)
>>   s.?('').?.?
>> end
>> string_shuffle("foobar")
=> "oobfra"
```

```ruby
代码清单 4.15: 添加到String类的shuffle函数的框架
>> class String
>>   def shuffle
>>     self.?('').?.?
>>   end
>> end
>> "foobar".shuffle
=> "borafo"
```

