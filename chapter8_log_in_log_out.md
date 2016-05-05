# 第八章 登录和退出

既然新用户能注册我们的网站了（第七章）。是时候让他们能登陆和退出了了。我们将为网站登陆和退出行为实现最常用的三个模型：“忘记”浏览器关闭的用户（8.1节和8.2节），自动记住用户（8.4节），依据“记住我”的复选框来选择性的记住用户（8.4.5节）。

我们开发这章的授权系统将允许我们自定义站点和依据登陆状态实现授权模型，失败当前用户。例如，在这章，我们将用登陆和退出链接更新站点头部和人物简介页面链接。在第九章，我们将强制只有登陆的用户能访问用户主页，仅仅正确的用户能进入页面编辑他们的信息，仅仅管理员能从数据库删除别的用户。最后，在第十一章，我们激昂十一登陆用户的识别来创建和用户关联的微博，在第十二章，我们将允许当前用户跟谁应用程序别的用户（因此收到他们微博的种子）。

这是一篇长的，覆盖许多关于常见登陆系统的细节的挑战性德一章，所以我推荐聚焦一节节的完成它。另外，许多读者报告说受益于再一次遍读。

## 8.1 会话

[HTTP](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)是[无状态协议](https://en.wikipedia.org/wiki/Stateless_protocol)，把每个请求都当做单独的交易，不能使用任何之前的请求。这意味着没有方法[在超文本传输协议里](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#HTTP_session_state)从页到页记住用户的身份；相反，网页应用程序要求用户登陆必须使用[会话](http://en.wikipedia.org/wiki/Session_(computer_science))，它是两台计算机之间半长久的连接（例如客户机运行网页服务器，服务器运行Rails）。

在Rails里最常见的实现会话的技术是使用[cookies](http://en.wikipedia.org/wiki/HTTP_cookie)，它是放在用户浏览器的小段文本。因为cookie在一个页面到下个页面一直存在，它们可以储存信息（例如用户id），这些信息可以被应用程序用来从数据库取回登陆用户。在这节和8.2节，我们将使用Rails方法，叫session来创建临时的会话，当浏览器关闭时会自动过期，然后在8.4节，我们将使用另一个名为cookies的Rails方法。

模型化会话作为REST的资源是方便的：访问登陆页面将为新建（new）会话渲染表格，登陆将创建（create）会话，退出将删除（destroy）它。不像用户资源，它使用数据库后端（通过User模型）来储存数据，会话资源将使用cookie，卷入登陆的许多工作来自于创建基于cookied的授权机制。在这节和下节，我们将通过构建会话控制器，登陆表格和相关的控制器动作来为这个工作做准备。我们在8.2节里完成用户登陆，通过添加必要的会话操作代码。

在先前的几章，我们在主题分支上做我们的工作，最后把改动合并至主分支：

```ruby
$ git checkout master
$ git checkout -b log-in-log-out
```
### 8.1.1 会话控制器

会话控制器登陆和退出相应的特殊的REST动作：登陆表格是通过new动作处理（在这节覆盖），事实上登陆是通过发送POST请求到create动作来处理（8.2节），退出是通过发送DELETE请求到destroy动作来处理（8.3节）。（回忆表7.1里HTTP动词和REST动作之间的关系。）

为了开始，我们生成带new动作的会话控制器：
```
$ rails generate controller Sessions new
```
(包含new实际上也生成了视图，它是为什么我们不包括像create和destroy这些没有相应视图的动作。）跟随7.2节注册网页，我们的计划是位创建新的会话创建登陆表格，如图8.1:
![图8.1：登陆表格的实体模型](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/login_mockup.png)

不像用户资源，它使用特殊的resources方法来自动包含完整的REST的路由（代码清单7.3），会话资源将仅使用命名的路由，处理GET和POST请求logn路由和DELETE请求logout路由。结果如代码清单8.1显示（它也会删除rails generate controller生成的不需要路由）。

```ruby
代码清单 8.1: Adding a resource to get the standard RESTful actions for sessions.
# config/routes.rb
 Rails.application.routes.draw do
  root                'static_pages#home'
  get    'help'    => 'static_pages#help'
  get    'about'   => 'static_pages#about'
  get    'contact' => 'static_pages#contact'
  get    'signup'  => 'users#new'
  get    'login'   => 'sessions#new'
  post   'login'   => 'sessions#create'
  delete 'logout'  => 'sessions#destroy'
  resources :users
end
```

在代码清单8.1定义的路由和URL相应的动作和那些用户的（表7.1）很相似，如表8.1显示。

HTTP request|URL|Named routes|Action|Purpose
# GET/|login|login_path|new|page for a new session (login)
# POST/|login|login_path|create|create a new session (login)
# DELETE/|logout|logout_path|destroy|delete a session (log out)
表8.1：代码清单8.1里会话提供的路由

因为我们现在添加了介个自定义命名路由，看我们程序完整的路由列表是有用的，我们能使用rake routes生成：
```
$ bundle exec rake routes
 Prefix Verb   URI Pattern               Controller#Action
     root GET    /                         static_pages#home
     help GET    /help(.:format)           static_pages#help
    about GET    /about(.:format)          static_pages#about
  contact GET    /contact(.:format)        static_pages#contact
   signup GET    /signup(.:format)         users#new
    login GET    /login(.:format)          sessions#new
          POST   /login(.:format)          sessions#create
   logout DELETE /logout(.:format)         sessions#destroy
    users GET    /users(.:format)          users#index
          POST   /users(.:format)          users#create
 new_user GET    /users/new(.:format)      users#new
edit_user GET    /users/:id/edit(.:format) users#edit
     user GET    /users/:id(.:format)      users#show
          PATCH  /users/:id(.:format)      users#update
          PUT    /users/:id(.:format)      users#update
          DELETE /users/:id(.:format)      users#destroy
```

理解结果的细节不必要，但是用这种方式浏览路由给我们支持的动作一个整体认识。

### 8.1.2 登陆表格
已经定义了相关的控制器和路由，现在我们将为新会话填充视图，例如，登陆表格。比较图8.1和图7.11，我们看见登陆表单和注册表单有点相似，除了用两个输入框（email和密码）代替了四个。

如图8.2所见，当登陆信息无效是我们想要重新渲染登陆也和现实错误的信息。在7.3.3节，我们使用错误信息片段来显示错误信息，但是我们看见在娜姐那些信息是有Active
Record自动提供的。这对创建会话的错误不适用，因为会话不是Active
Record对象，所以我们使用flash消息来渲染错误。

![图8.2：登陆失败的页面模型](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/login_failure_mockup.png)

从代码清单7.13回忆，注册表格使用form_for辅助方法，使用用户实例变量@user作为参数：
```ruby
<%= form_for(@user) do |f| %>
  .
  .
  .
<% end %>
```

会话表单和注册表单的主要不同是我们没有会话模型，因此没有对@user变量没有模拟。这意味着，在构建新会话表单，我们不得不给form_for更多点信息；具体来说，然而
```ruby
form_for(@user)
```
允许Rails推断表单的action应该是POST到URL
/users，在这个会话例子，我们需要表明资源的名称和相应的URL：
```ruby
form_for(:session, url: login_path)
```

随着正确的form_for在手，创建登陆表单来匹配图8.1的页面模型，使用注册表单（代码清单7.13）作为样板，如代码清单8.2所示：
```ruby
代码清单 8.2: Code for the login form.
# app/views/sessions/new.html.erb
 <% provide(:title, "Log in") %>
<h1>Log in</h1>

<div class="row">
  <div class="col-md-6 col-md-offset-3">
    <%= form_for(:session, url: login_path) do |f| %>

      <%= f.label :email %>
      <%= f.email_field :email, class: 'form-control' %>

      <%= f.label :password %>
      <%= f.password_field :password, class: 'form-control' %>

      <%= f.submit "Log in", class: "btn btn-primary" %>
    <% end %>

    <p>New user? <%= link_to "Sign up now!", signup_path %></p>
  </div>
</div>
```

注意，我们已经为了方便添加了注册页面的链接。有了代码清单8.2的代码，注册表格如图8.3所示。（因为“Log in”导航链接仍然没有填好，我不得不直接在地址栏输入/login URL，我们将在8.2.3节解决这个瑕疵。）

![图8.3：登陆表单](https://en.wikipedia.org/wiki/Stateless_protocol://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/login_form.png)

生成的表单的HTML如图代码清单8.3所示。

```html
<form accept-charset="UTF-8" action="/login" method="post">
  <input name="utf8" type="hidden" value="&#x2713;" />
  <input name="authenticity_token" type="hidden"
         value="NNb6+J/j46LcrgYUC60wQ2titMuJQ5lLqyAbnbAUkdo=" />
  <label for="session_email">Email</label>
  <input class="form-control" id="session_email"
         name="session[email]" type="text" />
  <label for="session_password">Password</label>
  <input id="session_password" name="session[password]"
         type="password" />
  <input class="btn btn-primary" name="commit" type="submit"
       value="Log in" />
</form>
```

比较代码清单8.3和7.15，你可能猜到提交表单将导致params哈希，有params[:session][:email]和prams[:session][:password]，和email和密码框对应。

### 8.1.3 查找和授权用户

和创建用户的例子里（注册）一样，在创建会话的（login）第一步是处理无效的输入。我们将通过复习当表单提交是发生了什么开始，然后为登陆失败的显示有用的错误信息做准备（如图8.2页面模型）然后我们将为成功的登陆奠定基础（8.2节）通过依据email和密码租的的有效性来评估每个登陆提交。

让我们从为会话控制器（Sessions controller）定义超简单的create动作，然后是空得new和destroy动作（代码清单8.4）。代码清单8.4里create动作除了渲染new视图，其他什么也没做，但是足够让我们开始。提交/sessions/new表单然后产生了如图8.4显示的结果。

```ruby
代码清单 8.4: A preliminary version of the Sessions create action.
# app/controllers/sessions_controller.rb
 class SessionsController < ApplicationController

  def new
  end

  def create
    render 'new'
  end

  def destroy
  end
end
```

![图8.4：初始化失败的登陆，带create如代码清单8.4](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/initial_failed_login_3rd_edition.png)

仔细查看图8.4里的调试信息显示，如在8.1.2节暗示的一样，提交导致一个名为params的哈希，在键session下有email和密码的组合，显示如下（忽略一些被Rails内部使用的一些不相干的细节）：

```ruby
---
session:
  email: 'user@example.com'
  password: 'foobar'
commit: Log in
action: create
controller: sessions
```
和用户注册的例子一样（图7.15），这些参数行程了内嵌哈希，想我们在代码清单4.10看到的。具体来说，params包含内嵌表单的哈希
```
{ session: { password: "foobar", email: "user@example.com" } }
```
这意味着
···
params[:session]
```
本身也是哈希：
```
{ password: "foobar", email: "user@example.com" }
```
结果，
···
params[:session][:email]
```
是提交的email地址，
···
params[:session][:password]
```
是提交的密码。

换句话说，在create动作里，params哈希有通过email和密码验证用户需要的所有信息。不是偶然的，我们已经有了确切地我们需要的方法：Active Record提供的User.find_by方法（6.1.4节）和has_secure_password提供的authenticate方法（6.3.4节）。回忆authenticate为无效的验证返回false（6.3.4节），我们为用户登陆的策略可以总结为如列表8.5所示。
```ruby
代码清单 8.5: Finding and authenticating a user.
# app/controllers/sessions_controller.rb
 class SessionsController < ApplicationController

  def new
  end

  def create
    user = User.find_by(email: params[:session][:email].downcase)
    if user && user.authenticate(params[:session][:password])
      # Log the user in and redirect to the user's show page.
    else
      # Create an error message.
      render 'new'
    end
  end

  def destroy
  end
end
```
在列表8.5里第一个高亮的行使用提交的email地址从数据库里取出用户。（回忆6.2.5节email地址被小写化以后保存，所有这里我们使用downcase方法，当提交的地址是有效的时，确保匹配。）下一行可能有点困惑，但是在Rails程序里是相当平常的语法：
```ruby
user && user.authenticate(params[:session][:password])
```

这使用&&（逻辑和）来决定是否结果用户是有效的。考虑在逻辑上下文里除了nil、false本身外任何对象都是true（4.2.3节），可能性如表8.2所示。我们从表8.2看见if语句是true，仅仅假如所给email的用户在数据库里有，而且所给的密码正是需要的。

UserPassworda && b
nonexistentanything(nil && [anything]) == false
valid userwrong password(true && false) == false
valid userwrongright password(true && true) == true
表8.2： user && user.authenciate(...)可能的结果

### 8.1.4带闪现信息渲染

回忆7.3.3节，我们使用用户模型的错误信息显示注册错误。这些错误是和特殊的Active
Record对象联系的，但是这个策略在这里不工作，因为回话不是Active
Record模型。相反，我们将在失败的登陆上显示一个闪现的消息。首先，有点不正确，的企图显示在代码清单8.6.

```
代码清单 8.6: An (unsuccessful) attempt at handling failed login.
# app/controllers/sessions_controller.rb
 class SessionsController < ApplicationController

  def new
  end

  def create
    user = User.find_by(email: params[:session][:email].downcase)
    if user && user.authenticate(params[:session][:password])
      # Log the user in and redirect to the user's show page.
    else
      flash[:danger] = 'Invalid email/password combination' # Not quite right!
      render 'new'
    end
  end

  def destroy
  end
end
```

因为显示在站点布局里的闪现消息（代码清单7.25），flash[:danger]信息自动显示；因为Bootstrap的CSS，它自动得到好看的样式化（图8.5）。

![图8.5：登陆失败的闪现消息](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/failed_login_flash_3rd_edition.png)

不幸的是，如在文本和在代码清单8.6里的注释里提到的，这段代码不是十分正确。页面看上去很好，不过，发生什么了？问题是flash的内容是持续一个请求，但是--不像重定向，我们在代码清单7.24里使用的--用render重新渲染模板不算请求。结果是flash信息显示的比我们想要的一个请求长。例如，假如我们提交无效的登陆信息，然后点击主页，flash又显示了（图8.6）。解决这个瑕疵是8.1.5的任务。

![图8.6：flash持续的例子](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/flash_persistence_3rd_edition.png)

### 8.1.5 flash测试
不正确的flash行为是我们应用程序的小bug。根据测试指导（旁注3.3），这恰好属于我们应该写测试来捕捉错误以便它不再发生的情况。我们继续前因此写为登陆表单写一个短的集成测试。另外，记录bug，然后防止回归，这也给我们好的基础，为将来进一步测试登陆和退出。

我们通过为我们的应用程序的登陆行为生成集成测试：
```
$ rails generate integration_test users_login
      invoke  test_unit
      create    test/integration/users_login_test.rb
```

接下来，我们需要测试捕捉显示在图8.5和8.6的序列。基本步骤如下：
1.访问登陆路径
2.确认新的会话表单正确的渲染
3.用一个无效的params哈希POST到会话路径
4.确认新会话表单再次渲染，flash信息显示
5.访问另一个页面（例如主页）
6.确认flash信息不显示在新页面。

以上步骤的测试实现显示在在代码清单8.7。

```
代码清单 8.7: A test to catch unwanted flash persistence. 红色
# test/integration/users_login_test.rb
 require 'test_helper'

class UsersLoginTest < ActionDispatch::IntegrationTest

  test "login with invalid information" do
    get login_path
    assert_template 'sessions/new'
    post login_path, session: { email: "", password: "" }
    assert_template 'sessions/new'
    assert_not flash.empty?
    get root_path
    assert flash.empty?
  end
end
```
添加测试到代码清单8.7以后，登陆测试应该是红色的：

```
代码清单 8.8: 红色
$ bundle exec rails test TEST=test/integration/users_login_test.rb
```
这显示了怎样运行一个（仅一个）测试文件，使用完整的文件路径。

让在列表8.7里的失败测试通过的方法是用flash.now来代替flash，flash.now是是设计用来在渲染过的页面上显示flash消息的。不像flash的内容，flash.now的内容只要有有一个另外的请求就消失了，这就是我们在列表8.7里要测试的行为。随着提交，相应的应用程序代码如列表8.9所示。

```
代码清单 8.9: Correct code for failed login. 绿色
# app/controllers/sessions_controller.rb
 class SessionsController < ApplicationController

  def new
  end

  def create
    user = User.find_by(email: params[:session][:email].downcase)
    if user && user.authenticate(params[:session][:password])
      # Log the user in and redirect to the user's show page.
    else
      flash.now[:danger] = 'Invalid email/password combination'
      render 'new'
    end
  end

  def destroy
  end
end
```
我们然后能确认登陆集成测试和完整的测试集是绿色的：

```
代码清单 8.10: 绿色
$ bundle exec rails test TEST=test/integration/users_login_test.rb
$ bundle exec rails test
```

## 8.2 登陆

既然我们的登陆表单很处理无效的提交，下一步是通过实际的用户登陆正确处理有效的提交。在这节，我们将用临时的会话cookie来登陆用户，当浏览器关闭时，会话自动失效。在8.4节，我们将添加持续性的会话，即使浏览器关闭也影响。

实现会话将需要定义大量的相关的函数，为使用多个控制器和视图。你可能回忆4.2.5节，Ruby提供了module工具，把这些函数打包在一个地方。方便地，当生成会话控制器时（8.1.1节）会话辅助方法模块也自动生成了。恶气，这样的辅助方法会自动包含在Rails视图里；通过包含模块进入所有控制器的基类（ApplicationController)，我们准备让它们在我们的控制器也工作（列表8.11）。

```
代码清单 8.11: Including the Sessions helper module into the Application
controller.
# app/controllers/application_controller.rb
 class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
  include SessionsHelper
end
```
随着配置结束，我们现在准备写登陆用户的代码。

### 8.2.1 log_in方法

有了Rails定义的session方法的帮助，让用户登陆是简单的。（这个方法是分开的，不同于8.1.1节生成的会话控制器）我们能对待session为哈希，可以使用如下方法赋值：

```
session[:user_id]=user.id
```
这在用户的浏览器放置了临时的cookie，包含加密了的用户的id，它允许我们在后续的网页使用session[:user_id]来获取id。相比cookie方法创建的持续性cookie（8.4节），session方法创建的cookie当浏览器关闭是立即过期。

因为我们想要使用同样的登陆技术在几个不同的地方，我将定义一个名为log_in的方法，在Session
helper里，如列表8.12显示。

```
代码清单 8.12: The log_in function.
# app/helpers/sessions_helper.rb
 module SessionsHelper

  # Logs in the given user.
  def log_in(user)
    session[:user_id] = user.id
  end
end
```

因为临时cookie创建使用了session方法自动加密，列表8.12里的代码是安全的，攻击者没有办法来使用登陆用户的会话信息。这仅仅应用到session方法初始化的临时会话，不过，不是使用cookies方法创建的持久性会话。持久性cookie是容易收到会话劫持攻击的，所以在8.4节，我们将不得不是更小心关于我们放在用户浏览器的信息。

随着我们在列表8.12里定义的log_in方法，我们现在准备完成会话create动作，通过登陆用户，然后重定向到用户的简介页面。结果如列表8.13显示。

```
代码清单 8.13: Logging in a user.
# app/controllers/sessions_controller.rb
 class SessionsController < ApplicationController

  def new
  end

  def create
    user = User.find_by(email: params[:session][:email].downcase)
    if user && user.authenticate(params[:session][:password])
      log_in user
      redirect_to user
    else
      flash.now[:danger] = 'Invalid email/password combination'
      render 'new'
    end
  end

  def destroy
  end
end
```
注意简短的重定向
```
redirect_to user
```
我们在7.4.1节之前见过。Rails自动把它转化称用户的简介页面的路由：
```
user_url(user)
```
随着在列表8.13里定义了create动作，在8.2节定义了登陆表单现在应该工作了。它在应用程序的显示上没有任何效果，不过，所有直接查看浏览器会话的缺点，没有方法分辨你登陆了。作为长更多可见变化的第一步，在8.2.2节我们将使用用户的id从数据库里获取当前用户的信息。在8.2.3节，我们将改变应用程序布局文件的链接，包括到当前用户的简介的URL。

### 8.2.2 当前用户

在临时会话里安全的放置了用户的id，我们现在到了在后续的网页获取它的时候了，我们将通过定义current_user方法来在数据库里找到与会话id相应的用户。current_user的目的是允许例如
```
<%= current_user.name %>
```
的结构和
```
redirect_to current_user
```
为了查找当前用户，一个可能性是使用find方法，如在用户简介页面上的（列表7.5）：
```
User.find(session[:user_id])
```

但是回忆6.1.4节，假如用户不存在，find抛出了异常。这个行为在用户简介页面是合适的，因为当id是无效的时候它才会发生，但是目前的情况是session[:user_id]将常常是nil（例如，没有登陆的用户）。为了处理这个可能性，我们将使用相同的find_by方法，过去在create动作里用来通过email地址查找用户的的find_by方法，用id代替email：

```
User.find_by(id: session[:user_id])
```
不是抛出例外，这个方法当id是无效的时候返回nil（表示没有这样的用户）。

我们现在能定义current_user方法如下：
```
def current_user
  User.find_by(id: session[:user_id])
end
```
这个将会很好的工作，但是它会使用几次数据库，假如，current_user在一个页面显示几次。相反，我们将使用常见的Ruby惯例，通过储存User.find_by的结果到一个实例变量，第一次它使用数据库，但是在后续的调用立即返回实例变量。
```
if @current_user.nil?
  @current_user = User.find_by(id: session[:user_id])
else
  @current_user
end
```

回忆在4.2.3节里见过的或操作付||，我们也能重写为：
```
@current_user = @current_user || User.find_by(id: session[:user_id])
```
因为User对象是在逻辑上下文里是true，对find_by的调用仅仅当@current_user仍然没有被赋值是执行。
尽管之前的代码将会工作，它不符合正确的Ruby的语言习惯；相反，正确给@current_user赋值的语法像这样：

```
@current ||= User.find_by(id: session[:user_id])
```
这里使用了容易造成困惑的，但是经常使用的||=（“或相等”）操作符（旁注8.1）

旁注8.1 *$@!是||=什么？
||=(“或等于”）赋值操作符是一个常见的Ruby语法，因此很重要，希望Rails程序员能认识。尽管刚开始它可能有点神秘，或等于通过模拟式容易理解的。

我们通过常见的变量递增模式开始：
x = x + 1
许多语言为这个操作提供了语法简写；在Ruby里（包括C, C++, Perl, Python,
Java等），它也能显示如下：

x += 1

模拟结构为别的操作符也存在：

$ rails console
>> x = 1
=> 1
>> x += 1
=> 2
>> x *= 3
=> 6
x -= 8
=> -2
>> x /= 2
=> -1

在每种例子，模式是x = x O y和 x O= y是对任何操作符O都是等价的。

另一个常见的Ruby模式是假如它是nil，赋值给变量，反之不变。回忆在4.2.3节看见的or操作符||，我们能这样写：
>> @foo
=> nil
>> @foo = @foo || "bar"
=> "bar"
>> @foo = @foo || "baz"
=> "bar"

因为nil在逻辑上下文是false，第一次赋值给@foo是nil ||
“bar”，最后的值为“bar”。相似地，第二次赋值是@foo || "baz"，例如，“bar” ||
“baz”，它的值也是“bar”。这是因为任何不适nil或false的在逻辑上下文都是true，系列||表达式在第一个为true的表达式结束。（这个计算的实践||表达式从左到右，然后在第一个值为true停下来，就是著名的短路求值。同样的原理也应用在&&语句，除了在这个情况下求值停在第一个值为false的位置。

比较控制台为几个操作符的会话，我们看见@foo = @foo || "bar"遵循 x = x O
y模式，用||代替了O：

  x    =   x   +   1      ->     x     +=   1
  x    =   x   *   3      ->     x     *=   3
  x    =   x   -   8      ->     x     -=   8
  x    =   x   /   2      ->     x     /=   2
  @foo = @foo || "bar"    ->     @foo ||= "bar"

因此我们看见 @foo = @foo || "bar" and @foo ||= "bar"
是等价的。在当前用户的上下文，这暗示下面的结构：

@current_user ||= User.find_by(id: session[:user_id])
Voilà !

(顺便提一下，在后台，Ruby真的计算表达式@foo || @foo =
"bar"，它避免了不必要的赋值，当@foo是nil或者false时。但是这个表达式也没有解释||=，所以以上的讨论使用了几乎等价的@foo
= @foo || "bar".)

应用了上报讨论的结果，产生了简明的current_user方法，显示在列表8.14.

```ruby
代码清单 8.14: Finding the current user in the session.
# app/helpers/sessions_helper.rb
 module SessionsHelper

  # Logs in the given user.
  def log_in(user)
    session[:user_id] = user.id
  end

  # Returns the current logged-in user (if any).
  def current_user
    @current_user ||= User.find_by(id: session[:user_id])
  end
end
```

有了在列表8.14里工作的current_user方法，我们现在到了依据用户登陆来改变一下我们的应用程序的位置了。

### 8.2.3 改变布局文件链接

第一个可用的登陆应用程序需要依据登陆状态改变布局文件链接。具体来说，如图8.7页面模型里所见，我们将为退出添加链接，为用户设置、为列出所有用户和为当前用户的简介页面。注意在途8.7里，退出和简介链接显示在下拉菜单“Account”里；我们将在列表8.16里看见怎样用Bootstrap创建这样的菜单。

![图8.7：成功登陆后的用户简介的页面模型](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/login_success_mockup.png)

在这点，在真实生活里，我会考虑写一个集成测试来捕捉上面描述的行为。如在旁注3.3里提到的，当你更熟悉Rails里的测试工具，你可能发现你自己更倾向于先写测试。在这个例子，不过，这样的测试需要几个新的想法，所有现在最好推迟到它自己的一节（8.2.4）。

改变网站布局里的链接的方法需要使用if-else语句，在内嵌的Ruby里面来显示一套链接，假如用户登陆和另一套相反的链接：
```ruby
<% if logged_in? %>
  # Links for logged-in users
<% else %>
  # Links for non-logged-in-users
<% end %>

```

这类型的代码需要logged_in?逻辑方法的存在，我们将定义它。

假如在会话里有当前用户，用户是登陆了的，例如，假如current_user不是nil。核对这个要求使用“不”操作符（4.2.3节），使用感叹号！,通常念“bang”。结果logged_in？方法显示在列表8.15里。

```ruby
代码清单 8.15: The logged_in? helper method.
# app/helpers/sessions_helper.rb
 module SessionsHelper

  # Logs in the given user.
  def log_in(user)
    session[:user_id] = user.id
  end

  # Returns the current logged-in user (if any).
  def current_user
    @current_user ||= User.find_by(id: session[:user_id])
  end

  # Returns true if the user is logged in, false otherwise.
  def logged_in?
    !current_user.nil?
  end
end
```

另外在列表8.15里，我们准备改变布局文件链接，假如用户登陆了。有四个新链接，两个是占位的（在第9章完成）：

```html
<%= link_to "Users",    '#' %>
<%= link_to "Settings", '#' %>
```
退出链接，同时，使用列表8.1定义的退出路径：
```ruby
<%= link_to "Log out", logout_path, method: :delete %>
```
注意登出链接传递哈希参数表明它应该带HTTP DELETE请求提交。我们也将添加简介链接如下：
```ruby
<%= link_to "Profile", current_user %>
```
这里我们能写成
```ruby
<%= link_to "Profile", user_path(current_user) %>
```
但是通常Rails允许我们直接链接到用户，通过自动转换current_user到user_path(current_user)在当前环境。最后，当用户没登陆，我们将使用列表8.1定义的登陆路径来创建一个到登陆表单的链接：
```ruby
<%= link_to "Log in", login_path %>
```
把这些放在一起给更新的头部片段显示在列表8.16.
```ruby
代码清单 8.16: Changing the layout links for logged-in users.
# app/views/layouts/_header.html.erb
 <header class="navbar navbar-fixed-top navbar-inverse">
  <div class="container">
    <%= link_to "sample app", root_path, id: "logo" %>
    <nav>
      <ul class="nav navbar-nav navbar-right">
        <li><%= link_to "Home", root_path %></li>
        <li><%= link_to "Help", help_path %></li>
        <% if logged_in? %>
          <li><%= link_to "Users", '#' %></li>
          <li class="dropdown">
            <a href="#" class="dropdown-toggle" data-toggle="dropdown">
              Account <b class="caret"></b>
            </a>
            <ul class="dropdown-menu">
              <li><%= link_to "Profile", current_user %></li>
              <li><%= link_to "Settings", '#' %></li>
              <li class="divider"></li>
              <li>
                <%= link_to "Log out", logout_path, method: "delete" %>
              </li>
            </ul>
          </li>
        <% else %>
          <li><%= link_to "Log in", login_path %></li>
        <% end %>
      </ul>
    </nav>
  </div>
</header>
```
作为包含新链接到布局文件的部分，列表8.16利用了Bootstrap的能力来创建下拉菜单。注意具体来说，特殊的Bootstrap
CSS类如dropdown，dropdown-menu，等等。为了激活下拉菜单，我们需要引入Bootstrap的自定义JavaScritp库，在Rails资产管理的application.js文件，如列表8.17所示：
```ruby
代码清单 8.17: Adding the Bootstrap JavaScript library to application.js.
# app/assets/javascripts/application.js
 //= require jquery
//= require jquery_ujs
//= require bootstrap
//= require turbolinks
//= require_tree .
```

在这点，你应该方法登陆路径，用有效用户登陆进去，它在先前的三节有效地测试了代码。有了列表8.16和8.17里的代码，你应该看见下拉菜单和登陆进去用户的链接，如图8.8所示。假如你完全退出浏览器，你应该也能确认应用程序忘记了你登陆的状态，要求你再次登陆来看见上面描述的变化。

![图8.8：带新链接和下拉菜单的登陆用户](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/profile_with_logout_link_3rd_edition.png)

### 8.2.4 测试布局变化
已经手动确认了应用程序在成功登陆后正确的响应，在我们继续前，我们将写一个集成测试来捕捉这个行为，然后捕捉回归。我们将从列表8.7构建这个测试，然后写一系列的步骤来确认接下来的动作：
1. 访问登陆路径
2. POST有效信息到会话路径
3. 确认登陆链接消失
4. 确认退出链接显示
5. 确认用户简介页面显示

为了看见这些变化，我们的测试需要作为知青注册用户登陆，这意味着这样的用户必须已经在数据库了。默认的Rails做这件事的方法是使用fixture，这是一种组织数据，加载到测试数据库的方式。我们在6.2.5节里发现我们需要删除默认的fixture以便我们的email唯一测试通过（列表6.30）。现在我们准备开始用我们自己的自定义fixture填充空文件。

在目前，我们仅需要一个用户，他得信息应该有有效的名字和email地址组成。因为我们需要让用户登陆，我们也不得不包含有效的密码来比较提交到会话控制器的create动作的密码一致。参考图6.8的数据模型，我们看见这意味着创建password_digest属性为用户fixture，我们通过定义我们自己的digest方法。

如在6.3.1节讨论的，密码摘要是使用bcrypt创建（通过has_secure_password)，所以我们需要使用相同的方法创建fixture密码。通过查看has_secure_password的源代码，我们发现这个方法是：
```ruby
BCrypt::Password.create(string, cost: cost)
```
string是已经hash的字符串，cost是cost参数，觉得计算哈希的计算花费。使用高的消费让使用哈希恢复原始密码更难，这对于在生产环境的安全预防措施是重要的，但是在测试里，我们想要digest方法尽可能的快。has_secure_password有一行也为这个：
```ruby
cost = ActiveModel::SecurePassword.min_cost ? BCrypt::Engine::MIN_COST :
                                              BCrypt::Engine.cost
```
这段有点晦涩的代码，你不需要理解细节，安排一下准确地描述上面的行为：它在测试里使用最小的花费的参数，和在生产环境里使用正常的（高的）花费。（我们将在8.4.5节学习更多关于奇怪的?-:表达式）

有几个different我们能放置digest方法，但是我们在8.4.1节有机会来在用户模型里重复使用digest。这暗示我们把它放在user.rb里。因为当我们计算摘要时我们不必进入用户对象（在fixture文件里也一样），我们将把digest方法附加到用户类本身，（如在4.4.1节里提过）把它作为类方法。结果显示在列表8.18里。

```ruby
代码清单 8.18: Adding a digest method for use in fixtures.
# app/models/user.rb
 class User < ActiveRecord::Base
  before_save { self.email = email.downcase }
  validates :name,  presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false }
  has_secure_password
  validates :password, presence: true, length: { minimum: 6 }

  # Returns the hash digest of the given string.
  def User.digest(string)
    cost = ActiveModel::SecurePassword.min_cost ? BCrypt::Engine::MIN_COST :
                                                  BCrypt::Engine.cost
    BCrypt::Password.create(string, cost: cost)
  end
end
```

随着列表8.18里的digest方法，我们现在准备创建为有效用户fixture，如列表8.19所示。
```ruby
代码清单 8.19: A fixture for testing user login.
# test/fixtures/users.yml
 michael:
  name: Michael Example
  email: michael@example.com
  password_digest: <%= User.digest('password') %>
```
注意fixture支持内嵌Ruby，它允许我们使用
```ruby
<%= User.digest('password') %>
```
来为测试用户创建有效地密码摘要。

尽管我们已经定义了has_secure_password需要的password_digest属性，有时也方便推断出实际密码。不幸的是，与fixture商定是不吭的，添加password属性到列表8.19引起Rails抱怨在数据库里没有这列（这是真的）。我们将通过采取所有的fixture用户有相同的密码（‘password’）。

已经用有效的用户创建了fixture，我们能在测试里取出它如下：
```ruby
user = users(:michael)
```
这里users和fixture文件名users.yml对应，然而在列表8.19里符号:michael使用键来参考用户。

有了上面的fixture用户，我们现在可以通过转换在这节开始后续的理解进入代码为布局文件的链接写测试，如列表8.20所示。

```ruby
代码清单 8.20: A test for user logging in with valid information. 绿色
# test/integration/users_login_test.rb
 require 'test_helper'

class UsersLoginTest < ActionDispatch::IntegrationTest

  def setup
    @user = users(:michael)
  end
  .
  .
  .
  test "login with valid information" do
    get login_path
    post login_path, session: { email: @user.email, password: 'password' }
    assert_redirected_to @user
    follow_redirect!
    assert_template 'users/show'
    assert_select "a[href=?]", login_path, count: 0
    assert_select "a[href=?]", logout_path
    assert_select "a[href=?]", user_path(@user)
  end
end
```
这里我们已经用了
```ruby
assert_redirected_to @user
```
来检查正确的重定向目标和
```ruby
follow_redirect!
```
实际访问目标页面。列表8.20也通过确认在网页上有零个登陆路径链接登陆链接确认它消失：
```ruby
assert_select "a[href=?]", login_path, count: 0
```
通过包含额外的count：
0选项，我们告诉assert_select我们期盼有零链接匹配所给的模式。（比较列表5.25里count:
2, 它检查恰好两个匹配链接。）

因为应用程序代码已经工作，测试应该是绿色的：
```ruby
代码清单 8.21: 绿色
$ bundle exec rails test TEST=test/integration/users_login_test.rb \
>                       TESTOPTS="--name test_login_with_valid_information"
```

这显示了怎样通过传递选项来运行测试文件里指定测试
```ruby
TESTOPTS="--name test_login_with_valid_information"
```
包含测试的名字。（测试的名字只是单词“test”和在测试描述一起用下划线连接。）注意第二行的>是“行持续”符号，是shell自动插入的，不应该手动输入。


### 8.2.5 注册登陆

尽管我们的授权系统现在工作了，新注册用户可能有点困惑，默认他们没有登陆。因为强迫用户注册后立即登陆是奇怪的，我们将让用户自动登陆作为注册过程的一部分。为了实现这个行为，所有要做的是在用户控制器create动作里添加调用log_in，如列表8.22显示。

```ruby
代码清单 8.22: Logging in the user upon signup.
# app/controllers/users_controller.rb
 class UsersController < ApplicationController

  def show
    @user = User.find(params[:id])
  end

  def new
    @user = User.new
  end

  def create
    @user = User.new(user_params)
    if @user.save
      log_in @user
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

为了测试列表8.22的行为，我们可以在列表7.26里加一行来检查用户已经登陆。在这个环境里在列表8.15里定义is_logged_in?辅助方法并行logged_in？辅助方法是有用的，假如在（测试）会话里有用户id返回true，否则返回false（列表8.23）。（因为辅助方法在测视里不可用，我们不能在列表8.15里使用current_users，但是session方法是可用的，所以我们将使用它。）这里我们使用is_logged_in?而不是logged_in?，以便测试辅助方法和Session辅助方法有不同的名字，这阻止彼此出错。
```ruby
代码清单 8.23: A boolean method for login status inside tests.
# test/test_helper.rb
 ENV['RAILS_ENV'] ||= 'test'
.
.
.
class ActiveSupport::TestCase
  fixtures :all

  # Returns true if a test user is logged in.
  def is_logged_in?
    !session[:user_id].nil?
  end
end
```

有了列表8.23里的代码，我们能断言用户使用显示在列表8.24里的那行在注册后登陆。
```ruby
代码清单 8.24: A test of login after signup. 绿色
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
    assert is_logged_in?
  end
end
```
在这点，测试集应该仍是绿色的：
```ruby
代码清单 8.25: 绿色
$ bundle exec rails test
```

## 8.3 退出

如在8.1节讨论的，我们的授权模型是让用户登陆，直到显示地退出。在这节，我们将添加这个必要的退出能力。因为“退出”链接已经被定义（列表8.16），所有我们要做的是写一个有效的控制器动作来销毁用户会话。

直到目前，Sessions
controller动作有如下REST的惯例，使用new为登陆页面，和create结束登录。我们通过使用destroy动作来删除会话继续这个主题，例如，退出。不像登陆函数，我们在列表8.13和8.22里使用，我们仅仅在一个地方退出，所有我们将把相关的代码直接放在destroy动作里。如在8.4.6节看见的，这个设计（带点重构）也将让授权机制更容易测试。

退出需要从列表8.12还原log_in方法的效果，需要从会话删除用户id。为了做这个，我们使用delete方法如下：
```ruby
session.delete(:user_id)
```

我们也将设置当前用户为nil，尽管在当前例子，这不重要，因为立即重定向到根URL。如在log_in和相关的方法，我们将把log_out方法放置在Sessions辅助方法模块，如列表8.26所示。
```ruby
代码清单 8.26: The log_out method.
# app/helpers/sessions_helper.rb
 module SessionsHelper

  # Logs in the given user.
  def log_in(user)
    session[:user_id] = user.id
  end
  .
  .
  .
  # Logs out the current user.
  def log_out
    session.delete(:user_id)
    @current_user = nil
  end
end
```

我们能吧log_out方法放在会话控制器的destroy动作了i列表8.27所示。
```ruby
代码清单 8.27: Destroying a session (user logout).
# app/controllers/sessions_controller.rb
 class SessionsController < ApplicationController

  def new
  end

  def create
    user = User.find_by(email: params[:session][:email].downcase)
    if user && user.authenticate(params[:session][:password])
      log_in user
      redirect_to user
    else
      flash.now[:danger] = 'Invalid email/password combination'
      render 'new'
    end
  end

  def destroy
    log_out
    redirect_to root_url
  end
end
```

为了测试退出机制，我们能从列表8.20添加一些步骤到用户登陆测试。登陆后，我们使用delete来发出一个DELETE请求到退出路径（表8.1）和确认用户退出，重定向到根URL。我们也检查登陆链接重现，退出和简介链接消失。新的步骤显示在列表8.28里。
```ruby
代码清单 8.28: A test for user logout. 绿色
# test/integration/users_login_test.rb
 require 'test_helper'

class UsersLoginTest < ActionDispatch::IntegrationTest
  .
  .
  .
  test "login with valid information followed by logout" do
    get login_path
    post login_path, session: { email: @user.email, password: 'password' }
    assert is_logged_in?
    assert_redirected_to @user
    follow_redirect!
    assert_template 'users/show'
    assert_select "a[href=?]", login_path, count: 0
    assert_select "a[href=?]", logout_path
    assert_select "a[href=?]", user_path(@user)
    delete logout_path
    assert_not is_logged_in?
    assert_redirected_to root_url
    follow_redirect!
    assert_select "a[href=?]", login_path
    assert_select "a[href=?]", logout_path,      count: 0
    assert_select "a[href=?]", user_path(@user), count: 0
  end
end
```
(既然我们有is_logged_in?在测试里可用，我们在POST有效信息到会话路径后也被立即抛给assert_is_logged_in?福利。）

随着会话destroy动作，因此被定义和测试，初始注册/登陆/退出三人小组结束了，测试集应该是绿色的：
```ruby
代码清单 8.29: 绿色
$ bundle exec rails test
```

## 8.4 记住我
我们在8.2节完成的登陆系统是自包含的和功能完整的，但是许多网站有额外的能力来记住用户的会话，即使他们关闭了浏览器。在这节，我们将开始默认记住用户登陆，仅仅当他们显示退出时国企他们的会话。在8.4.5节，我们将启用一般改变的模型，“记住我”复选框允许用户选择记住。这两个模型是专家级的，首先被Github和Bitbucket使用，然后Facebook和Twitter也开始使用。

### 8.4.1 记住口令和摘要

在8.2节，我们使用Rails的session方法来储存用户的id，但是这个信息当用户关闭浏览器后就消失了。在这节，我们将首先通过生成记住口令来使用cookie方法来创建持久性cookie，和为了验证那些口令安全的记住摘要一起。

如同在8.2.1节里提到的，使用session储存的信息是自动安全的，但是这不是说使用cookie储存的信息。具体来说，持久性cookie容易受到会话劫持，攻击者可以使用偷来的记住口令来作为特殊的用户来登陆。有四个主要的方法来偷cookie：（1）使用包监听器发现在不安全网络传输的cookie，（2)下载了包含记住口令的数据库，（3）使用跨站脚本（XSS），和（4）获得登陆用户的物理机器进入权限。我们在7.5节通过使用全站安全套接层（SSL），保护数据不被包监听器监听。我们使用储存记住口令的哈希摘要来防止第二种问题，和在6.3节另一个同样的方式，我们不是储存密码而是储存密码摘要。Rails通过转义插入视图的任何内容自动阻止第三个问题。最后，尽管没有铁一般的方法来阻止物理进入登陆者的计算机，我们将通过每次用户退出都改名口令和细心地加密任何潜在第敏感信息，我们防止在浏览器。

有了这些设计和安全考虑在心，我们创建持久会话的计划显现如下：
1. 创建随机的数字字符串作为记住口令
2. 把口令放在浏览器cookie，用离现在很远的日子作为过期日期
3. 保存哈希摘要到数据库
4. 在浏览器放置加密版的用户id
5.
当包含持久用户id的cookie出现，使用所给id在数据库里查询用户，然后确认记住口令cookie和数据库里的相关的哈希摘要匹配

注意最后一步登陆用户是多么相似，我们通过email地址取回用户，然后确认（使用authenticate方法）提交密码匹配密码摘要（列表8.5）。结果，我们的实现将并行has_secure_password的方面。

我们将通过田间需要的remember_digest属性到用户模型，如图8.9所示：

![图8.9：添加remember_digest属性的用户模型](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_model_remember_digest.png)

为了从图8.9添加数据模型到我们的应用程序，我们将生成一个数据迁移：
```ruby
$ rails generate migration add_remember_digest_to_users remember_digest:string
```
(比较在6.3.1节里的密码摘要数据迁移。）如在之前的迁移里，我们使用了以_to_users结尾的迁移名称来告诉Rails数据迁移被设计用来改变数据库里的users表。因为我们也包含了属性（remember_digest)和类型（string），Rails为我们生成了默认的迁移，如列表8.30所示。

```ruby
代码清单 8.30: The generated migration for the remember digest.
# db/migrate/[timestamp]_add_remember_digest_to_users.rb
 class AddRememberDigestToUsers < ActiveRecord::Migration
  def change
    add_column :users, :remember_digest, :string
  end
end
```
因为我们不准备通过使用记住摘要来取出用户，所以没有必要在remember_digest列上添加索引，我们可以使用上面生成的默认迁移：
```ruby
$ bundle exec rake db:migrate
```

现在我们不得不决定使用记住口令做什么了。有许多等价的可能性--必要地，任何长度的字符串将做。urlsafe_base64方法，来自于Ruby标准库的SecureRandom适合这个：它返回随机的长度为22的由A-Z，a-z,0-9,"-",和"_"(总共64种可能性，因此叫“base64”）组成的字符串）。典型的base64字符串显示如下：
```ruby
$ rails console
>> SecureRandom.urlsafe_base64
=> "q5lt38hQDc_959PVoo6b7A"
```
假如两个用户有相同的密码，完全没事，记住口令不必是唯一的，但是假如它们是唯一的话，会更安全。在base64的例子，22个字符每一位有64种可能性，所以俩个记住口令相撞的可能性是可以忽略不计的。作为福利，使用专门设计在URL里为了安全使用base64字符串（名字urlsafe_base64显示的），我们将使用同样的口令生成器来在第十章考虑账户激活和密码重置链接。

记住用户需要创建记住口令和保存口令摘要到数据库。我们已经定义了digest方法，为在测试fixture里使用（列表8.18），我们能使用上面讨论的创建new_tocke方法来创建新口令。和digest医院，新口令方法不需要用户对象，所以我们将让它成为类方法。结果是列表8.31显示的用户模型。
```ruby
class User < ActiveRecord::Base
  before_save { self.email = email.downcase }
  validates :name,  presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false }
  has_secure_password
  validates :password, presence: true, length: { minimum: 6 }

  # Returns the hash digest of the given string.
  def User.digest(string)
    cost = ActiveModel::SecurePassword.min_cost ? BCrypt::Engine::MIN_COST :
                                                  BCrypt::Engine.cost
    BCrypt::Password.create(string, cost: cost)
  end

  # Returns a random token.
  def User.new_token
    SecureRandom.urlsafe_base64
  end
end
```

我们为实现这个的计划是创建user.remember方法，和用户相关联的记住口令，保存相应的记住摘要到数据库。因为在列表8.30里的迁移，用户模型已经有一个remember_digest属性，但是仍然没有remember_token属性。我们需要一个通过user.remember_token来创建口令的方法（为了在cookie里保存），不保存它在数据库里。我们解决相似的问题用安全密码在6.3节，它在数据库使用了password属性和安全的password_digest属性。在那个情况，虚拟的属性自动被has_secure_password创建，但是我们将不得不为remember_token写代码。做这个事使用attr_accessor来创建可读取的属性，我们在4.4.5节里见过：
```ruby
class User < ActiveRecord::Base
  attr_accessor :remember_token
  .
  .
  .
  def remember
    self.remember_token = ...
    update_attribute(:remember_digest, ...)
  end
end
```
注意在remember方法里第一行赋值。因为Ruby在对象里吹赋值，没有self会创建一个名为remember_token的局部变量，这不是我们想要的。使用self确保赋值给用户的remember_token属性。（现在你指定为什么列表6.31里before_save回叫使用self.email而不是email）同时，第二行remember使用update_attribute方法来更新记住摘要。（如在6.1.5节里提到的，这个方法通过验证，在这里是必要的，因为我们不需要读写用户的密码或者确认信息。）

有了这些考虑，我们能创建有效的口令和使用通过首先使用User.new_token来创建新的记住口令，然后更新记住摘要用英语User.digest。这个过程给出remember方法，显示在列表8.32里。
```ruby
代码清单 8.32: Adding a remember method to the User model. 绿色
# app/models/user.rb
 class User < ActiveRecord::Base
  attr_accessor :remember_token
  before_save { self.email = email.downcase }
  validates :name,  presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false }
  has_secure_password
  validates :password, presence: true, length: { minimum: 6 }

  # Returns the hash digest of the given string.
  def User.digest(string)
    cost = ActiveModel::SecurePassword.min_cost ? BCrypt::Engine::MIN_COST :
                                                  BCrypt::Engine.cost
    BCrypt::Password.create(string, cost: cost)
  end

  # Returns a random token.
  def User.new_token
    SecureRandom.urlsafe_base64
  end

  # Remembers a user in the database for use in persistent sessions.
  def remember
    self.remember_token = User.new_token
    update_attribute(:remember_digest, User.digest(remember_token))
  end
end
```
### 8.4.2 使用记住登陆

已经创建了可用的user.remember方法，我们现在到了通过储存用户的（加密的）id和记住口令作为浏览器里永久cookie创建永久会话。做这个的方法是用cookies方法，它（和session一样）我们可以当做哈希。cookie包含两组信息，value和可选的expires日期。例如，我们能创建永久会话通过创建和值等价的cookie来记住在20年后过期的记住口令：
```ruby
cookies[:remember_token] = { value:   remember_token,
                             expires: 20.years.from_now.utc }
```
（这使用Rails时间辅助方法的管理，在旁注8.2节里讨论的。）这个设置cookie将在20年后过期的模式在Rails里是很普遍，所以有个特殊的permanent方法来实现它，所以我们可以简写为：
```ruby
cookies.permanent[:remember_token] = remember_token
```
这引起Rails自动设置为20.years.from_now。

旁注8.2 Cookie在20.years.from_now过期

你可能回忆4.4.2节Ruby让你添加方法到任何类，甚至内建类。在那节，我们把palindrom？方法添加到String类（结果发现“deified”是回文），我们也看见Rails怎样添加blank?方法到Object（所以"".blank?," ".blank?, 和nil.blank?都是true）。cookies.permanent方法，用20.years.from_now创建了“永久”cookie，仍然给了这个实践通过一个Rails时间辅助方法，这些方法被加到Fixnum类（整数的基类）：
```ruby
$ rails console
>> 1.year.from_now
  => Sun, 09 Aug 2015 16:48:17 UTC +00:00
  >> 10.weeks.ago
  => Sat, 31 May 2014 16:48:45 UTC +00:00

```
Rails也添加了别的辅助方法：
```ruby
>> 1.kilobyte
  => 1024
  >> 5.megabytes
  => 5242880
```
这些对上传验证有效，使得限制容易，例如，图片不超过5.magebytes。

尽管它应该小心使用，添加方法到内建类的弹性允许自然地扩展Ruby。确实，许多优雅的Rails起源于Ruby语言的扩展性。

为了在cookie里保存用户的id，我们能跟随使用session方法的模式（列表8.12）使用像
```ruby
cookies[:user_id] = user.id
```
因为它放置id作为普通的文本，这个方法暴露了应用程序的cookie的表单和为攻击者盗取用户的账户更容易。为了避免这个问题，我们使用署名cookie，它会在放到浏览器前安全地加密cookie：
```ruby
cookies.signed[:user_id] = user.id
```
因为我们想要用户id和永久记住口令配对，我们应该让它也永久，我们可以通过链接signed和permanent方法：
```ruby
cookies.permanent.signed[:user_id] = user.id
```
cookie被设置后，在后续页面视图里我们能像这样取回代码：
```ruby
User.find_by(id: cookies.signed[:user_id])
```
cookies.signed[:user_id]自动解密用户id
cookie。我们然后可以使用bcrypt来确认cookies[:remember_token]匹配在列表8.32里生成的remember_digest。（万一你正考虑为什么我们不使用署名用户id，没有记住口令，这会允许拥有加密id的攻击者作为合法用户永久登陆。在目前的设计，攻击者需要两个cookie才能登陆，仅仅持续到用户退出。）

最后困惑的是确认所给的记住口令匹配用户的记住摘要，在这个环境里，有几个等价的方法来使用bcrypt来确认匹配。假如你看[安全密码源代码](https://github.com/rails/rails/blob/master/activemodel/lib/active_model/secure_password.rb)你会看见像这样的比较：
```ruby
BCrypt::Password.new(password_digest) == unencrypted_password
```
在我们的例子，模拟代码将看起来像这个：
```ruby
BCrypt::Password.new(remember_digest) == remember_token
```
假如你想想它，这段代码是真正奇怪：它显示了直接和口令比较bcrypt密码摘要，暗示为了使用==比较解密了摘要。但是使用bcrypt的整个要点是不可逆，所以这不可能。确实，深挖[bcrypt gem源代码](https://github.com/codahale/bcrypt-ruby/blob/master/lib/bcrypt/password.rb)确认比较符==是重定义了，在后台上面的比较是和接下来等价：
```ruby
BCrypt::Password.new(remember_digest).is_password?(remember_token)
```
和==相反，这使用了逻辑方法is_password?来执行比较。因为它的意思有点清晰，我们在应用程序里更偏好第二种比较。

上面的讨论暗示放摘要-口令比较进入用户模型里authenticated?方法，它扮演has_secure_password提供的authenticate方法相似的角色（列表8.13）。实现显示在在8.33里。（尽管在列表8.33里authenticated?方法是指明了记住摘要，它将证明在别的环境也是有用的，我们将在第十章里一般化它）
```ruby
代码清单 8.33: Adding an authenticated? method to the User model.
# app/models/user.rb

class User < ActiveRecord::Base
  attr_accessor :remember_token
  before_save { self.email = email.downcase }
  validates :name, presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z]\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false }
  has_secure_password
  validates :password, presence: true, length: { minimum: 6 }

  # 返回所给字符串的哈希摘要
  def User.digest(string)
    cost = ActiveModel::SecurePassword.min_cost ? BCrypt::Engine::MIN_COST :
                                                  BCrypt::Engine.cost
    BCrypt::Password.create(string, cost: cost)
  end

  # Returns a random token.
  def User.new_token
    SecureRandom.urlsafe_base64
  end

  # Remembers a user in the database for use in persistent sessions.
  def remember
    self.remember_token = User.new_token
    update_attribute(:remember_digest, User.digest(remember_token))
  end

  # Returns true if the given token matches the digest.
  def authenticated?(remember_token)
    BCrypt::Password.new(remember_digest).is_password?(remember_token)
  end
end
```

注意定义在列表8.33里的在authenticated?里的remember_token参数和我们在列表8.32里使用的attr_accessor :remember_token不一样；相反，它是方法里的局部变量。（因为推断记住口令参数，它不是不普通的使用方法参数一样的名字。）也注意remember_digest属性，它和self.remember_digest是一样的，像在第六章里的name和email，自动被Active Record依据（列表8.30）相应的数据库里的列对应的。

我们现在到了记住登陆用户的位置，我们将通过添加remember辅助方法来和log_in一道，如列表8.34显示。

```ruby
代码清单 8.34: Logging in and remembering a user.
# app/controllers/sessions_controller.rb
 class SessionsController < ApplicationController

  def new
  end

  def create
    user = User.find_by(email: params[:session][:email].downcase)
    if user && user.authenticate(params[:session][:password])
      log_in user
      remember user
      redirect_to user
    else
      flash.now[:danger] = 'Invalid email/password combination'
      render 'new'
    end
  end

  def destroy
    log_out
    redirect_to root_url
  end
end
```
和log_in一样，列表8.34延迟了会话辅助方法的真正工作，我们定义remember方法，调用user.remember，因此生成记住口令和把摘要保存到数据库。它然后使用cookies来为用户id创建如上面所说的永久cookie和记住口令。结果显示在列表8.35。

```ruby
代码清单 8.35: Remembering the user.
# app/helpers/sessions_helper.rb
 module SessionsHelper

  # Logs in the given user.
  def log_in(user)
    session[:user_id] = user.id
  end

  # Remembers a user in a persistent session.
  def remember(user)
    user.remember
    cookies.permanent.signed[:user_id] = user.id
    cookies.permanent[:remember_token] = user.remember_token
  end

  # Returns the current logged-in user (if any).
  def current_user
    @current_user ||= User.find_by(id: session[:user_id])
  end

  # Returns true if the user is logged in, false otherwise.
  def logged_in?
    !current_user.nil?
  end

  # Logs out the current user.
  def log_out
    session.delete(:user_id)
    @current_user = nil
  end
end
```

有了列表8.35里的代码，用户登陆将在它们的浏览器得到有效的记住口令时记住，但是因为current_user方法定义在列表8.14里的仅知道临时会话，所以仍然对我们没有任何用处：
```ruby
@current_user ||= User.find_by(id: session[:user_id])
```
在永久会话的情形下，假如session[:user_id]存在，我们想要从临时会话取回用户，但是否则我们应该查找cookies[:user_id]来取回（和登陆）用户相应永久会话。我们能完成这个如下：
```
if session[:user_id]
  @current_user ||= User.find_by(id: session[:user_id])
elsif cookies.signed[:user_id]
  user = User.find_by(id: cookies.signe[:user_id])
  if user && user.authenticated?(cookies[:remember_token])
    log_in user
    @current_user = user
  end
end
```
(这和列表8.5里看见的user &&
user.authenticated模式是一样的。）上面的代码会工作，但是注意session和cookies重复使用。我们能消除重复如下：
```
if (user_id = session[:user_id])
  @current_user ||= User.find_by(id: user_id)
elsif (user_id = cookies.signed[:user_id])
  user = User.find_by(id: user_id)
  if user && user.authenticated?(cookies[:remember_token])
    log_in users
    @current_user = user
   end
end
```

这使用了相同但是潜在让人困惑的结构
```ruby
if (users_id = session[:user_id])
```

尽管显示，这不是比较（比较使用两个等号==），而是赋值。假如你按照语言读它，你不会说“假如用户id等于会话的用户id...”，而是像“假如会话的用户id存在（当设置用户id到会话的用户id时）。。。”

定义current_user辅助方法如上面讨论的，导致如列表8.36所示的实现。

```ruby
代码清单 8.36: Updating current_user for persistent sessions. 红色
# app/helpers/sessions_helper.rb
 module SessionsHelper

  # Logs in the given user.
  def log_in(user)
    session[:user_id] = user.id
  end

  # Remembers a user in a persistent session.
  def remember(user)
    user.remember
    cookies.permanent.signed[:user_id] = user.id
    cookies.permanent[:remember_token] = user.remember_token
  end

  # Returns the user corresponding to the remember token cookie.
  def current_user
    if (user_id = session[:user_id])
      @current_user ||= User.find_by(id: user_id)
    elsif (user_id = cookies.signed[:user_id])
      user = User.find_by(id: user_id)
      if user && user.authenticated?(cookies[:remember_token])
        log_in user
        @current_user = user
      end
    end
  end

  # Returns true if the user is logged in, false otherwise.
  def logged_in?
    !current_user.nil?
  end

  # Logs out the current user.
  def log_out
    session.delete(:user_id)
    @current_user = nil
  end
end
```
有了列表8.36里的代码，新的登陆用户被正确地记住了，你可以通过登陆确认，关掉浏览器，然后检查当你重启示例应用程序然后重新方法示例程序。假如你像，你甚至可以查看浏览器cookie直接看看结果（图8.10）

![图8.10：在本地浏览器的记住口令](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/cookie_in_browser_chrome.png)

现在仅仅有一个问题：缺少清除他们的浏览器cookie（或者等20年），没有用户退出的方法。这恰好是我们测试集应该捕捉的，确实它现在应该是红色的：
```ruby
$ bundle exec rails test
```

### 8.4.3 忘记用户
为了允许用户退出，我们将定义忘记用户的方法来在模拟记住他们里。结果user.forget方法只是还原user.remember通过更行记住摘要为nil，如列表8.38所示。
```ruby
代码清单 8.38: Adding a forget method to the User model.
# app/models/user.rb
 class User < ActiveRecord::Base
  attr_accessor :remember_token
  before_save { self.email = email.downcase }
  validates :name,  presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false }
  has_secure_password
  validates :password, presence: true, length: { minimum: 6 }

  # Returns the hash digest of the given string.
  def User.digest(string)
    cost = ActiveModel::SecurePassword.min_cost ? BCrypt::Engine::MIN_COST :
                                                  BCrypt::Engine.cost
    BCrypt::Password.create(string, cost: cost)
  end

  # Returns a random token.
  def User.new_token
    SecureRandom.urlsafe_base64
  end

  # Remembers a user in the database for use in persistent sessions.
  def remember
    self.remember_token = User.new_token
    update_attribute(:remember_digest, User.digest(remember_token))
  end

  # Returns true if the given token matches the digest.
  def authenticated?(remember_token)
    BCrypt::Password.new(remember_digest).is_password?(remember_token)
  end

  # Forgets a user.
  def forget
    update_attribute(:remember_digest, nil)
  end
end
```
有了在列表8.38里的代码，我们现在准备忘记永久会话通过添加forget辅助方法，然后在log_out辅助方法里调用它（列表8.39）。如在列表8.39里，forget辅助方法调用user.forget，然后删除user_id和remember_token cookie。
```ruby
代码清单 8.39: Logging out from a persistent session.
# app/helpers/sessions_helper.rb
 module SessionsHelper

  # Logs in the given user.
  def log_in(user)
    session[:user_id] = user.id
  end
  .
  .
  .
  # Forgets a persistent session.
  def forget(user)
    user.forget
    cookies.delete(:user_id)
    cookies.delete(:remember_token)
  end

  # 退出当前用户
  def log_out
    forget(current_user)
    session.delete(:user_id)
    @current_user = nil
  end
end
```

### 8.4.4 两个细微的bug
有两个很相近的细微之处留下来说明。第一个是即使仅仅当登陆是，“退出”链接显示，用户能潜在第让多个浏览器窗口打开网站。假如用户在一个窗口登出，在第二个窗口点击“退出”链接将会导致错误，由于在列表8.39里current_user的使用。我们能通过仅仅假如用户登陆时退出来避免这个。

第二个细微之处是用户可能登陆（和记住）在几个浏览器，例如Chrome和Firefox，这引起一个问题，假如用户在一个浏览器退出，但是没有在其他。例如，设想用户在Firefox里退出，因此设置记住摘要为nil（通过列表8.38里的user.forget)。应用程序将在Firefox仍然工作；因为log_out方法在列表8.39里删除了用户的id，两个高亮的条件是false：
```ruby
# Returns the user corresponding to the remember token cookie.
def current_user
  if (user_id = session[:user_id])
    @current_user ||= User.find_by(id: user_id)
  elsif (user_id = cookies.signed[:user_id])
    user = User.find_by(id: user_id)
    if user && user.authenticated?(cookies[:remember_token])
      log_in user
      @current_user = user
    end
  end
end
```

结果，求值到了current_user方法的最后，因此如要求的返回nil。

相反，在Chrome里，user_id
cookie还没有被删除，所以user_id将是true，在逻辑环境里，和相应的用户将在数据库里被发现：

```ruby
# Returns the user corresponding to the remember token cookie.
def current_user
  if (user_id = session[:user_id])
    @current_user ||= User.find_by(id: user_id)
  elsif (user_id = cookies.signed[:user_id])
    user = User.find_by(id: user_id)
    if user && user.authenticated?(cookies[:remember_token])
      log_in user
      @current_user = user
    end
  end
end
```
因此，内部的if条件将会执行：
```ruby
 user && user.authenticated?(cookies[:remember_token])
```

具体来说，因为user不是nil，第二个表达式将被执行，它会抛出错误。这是因为用户的记住摘要作为退出的部分被删除在Firefox里（列表8.38），所以当我们在Chrome里进入应用程序，我们结束调用
```
BCrypt::Password.new(remember_digest).is_password?(remember_token)
```
带nil记住摘要，因此在bcrypt库内部抛出错误。为了解决这个，我们想要authenticated?返回false。

这些恰好是受益于测试驱动开发的细微之处，所以我们将在改正它们前写测试来捕捉两种错误。我们首先让列表8.28列的集成测试为红色，如立白8.40显示。

```ruby
代码清单 8.40: A test for user logout. 红色
# test/integration/users_login_test.rb
 require 'test_helper'

class UsersLoginTest < ActionDispatch::IntegrationTest
  .
  .
  .
  test "login with valid information followed by logout" do
    get login_path
    post login_path, session: { email: @user.email, password: 'password' }
    assert is_logged_in?
    assert_redirected_to @user
    follow_redirect!
    assert_template 'users/show'
    assert_select "a[href=?]", login_path, count: 0
    assert_select "a[href=?]", logout_path
    assert_select "a[href=?]", user_path(@user)
    delete logout_path
    assert_not is_logged_in?
    assert_redirected_to root_url
    # 模拟用户在另个窗口点击退出
    delete logout_path
    follow_redirect!
    assert_select "a[href=?]", login_path
    assert_select "a[href=?]", logout_path,      count: 0
    assert_select "a[href=?]", user_path(@user), count: 0
  end
end
```

另一个在列表8.40里的调用delete
logout_path应该抛出错误，由于缺少current_user，导致测试集为红色：
```ruby
代码清单 8.41: 红色
$ bundle exec rails test
```
应用程序代码仅仅在logge_in?为真时调用log_out，如列表8.42显示：
```ruby
代码清单 8.42: Only logging out if logged in. 绿色
# app/controllers/sessions_controller.rb
 class SessionsController < ApplicationController
  .
  .
  .
  def destroy
    log_out if logged_in?
    redirect_to root_url
  end
end
```
另一个情形，需要两个不同的浏览器的场景，用集成测试更难模拟，但是直接测试用户模型是容易的。所有我们需要做的是用没有记住摘要的用户喀什（对于在setup里定义的@user变量来说是真），然后调用authenticated?，如列表8.34显示。（注意我们只是留下记住口令为空；它的值是什么不重要，因为在使用它前错误已经发生。）

```
代码清单 8.43: A test of authenticated? with a nonexistent digest. 红色
# test/models/user_test.rb
 require 'test_helper'

class UserTest < ActiveSupport::TestCase

  def setup
    @user = User.new(name: "Example User", email: "user@example.com",
                     password: "foobar", password_confirmation: "foobar")
  end
  .
  .
  .
  test "authenticated? should return false for a user with nil digest" do
    assert_not @user.authenticated?('')
  end
end
```
因为BCrypt::Password.new(nil)抛出错误，测试集现在应该是红色的：
```
代码清单 8.44: 红色
$ bundle exec rails test
```
为了解决这个错误，让它变绿，所有我们需要做的是返回false，假如记住摘要是nil，如列表8.45所示。

```ruby
代码清单 8.45: Updating authenticated? to handle a nonexistent digest. 绿色
# app/models/user.rb
 class User < ActiveRecord::Base
  .
  .
  .
  # 假如所给的口令匹配摘要
  def authenticated?(remember_token)
    return false if remember_digest.nil?
    BCrypt::Password.new(remember_digest).is_password?(remember_token)
  end
end
```

假如记住摘要是nil，这里使用return关键词来立即返回，这是普通方法来强调方法剩余部分在那种情形被忽略。等价的代码
```ruby
if remember_digest.nil?
  false
else
  BCrpyt::Password.new(remember_digest).is_password?(remember_token)
end
```
也会很好的工作，但是我偏好列表8.45里的版本（也碰巧更短一点）。

有力列表8.45里的代码，我们完整的测试集应该是绿色的，两个细微的bug应该现在被表明了：
```ruby
$ bundle exec rails test
```

### 8.4.5 “记住我”复选框

有了8.4.4节的代码，我们的应用程序有了完整的，专家级的授权系统。作为最后一步，我们将看见怎样使用“记住我”复选框保持登陆。带复选框的登陆表单显示在途8.11。

![图8.11：“记住我”复选框的页面模型](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/login_remember_me_mockup.png)

为了写实现代码，我们通过从列表8.2里添加复选框到登陆表单开始。和标签、文本框、密码框和提交按钮等一样，复选框可以通过Rails辅助方法创建。为了让样式正确，不过，我们不得不吧复选框嵌套进标签里，如下：

```html
<%= f.label :remember_me, class: "checkbox inline" do %>
  <%= f.check_box :remember_me %>
  <span>Remember me on this computer</span>
<% end %>
```
把这个放进登陆表单，代码如列表8.47所示。

```html
代码清单 8.47: Adding a “remember me” checkbox to the login form.
# app/views/sessions/new.html.erb
 <% provide(:title, "Log in") %>
<h1>Log in</h1>

<div class="row">
  <div class="col-md-6 col-md-offset-3">
    <%= form_for(:session, url: login_path) do |f| %>

      <%= f.label :email %>
      <%= f.email_field :email, class: 'form-control' %>

      <%= f.label :password %>
      <%= f.password_field :password, class: 'form-control' %>

      <%= f.label :remember_me, class: "checkbox inline" do %>
        <%= f.check_box :remember_me %>
        <span>Remember me on this computer</span>
      <% end %>

      <%= f.submit "Log in", class: "btn btn-primary" %>
    <% end %>

    <p>New user? <%= link_to "Sign up now!", signup_path %></p>
  </div>
</div>
```
在列表8.47里，我们已经包含了CSS类checkbox和inline，它们是Bootstrap用来把复选框和文本（“在这台计算机上记住我”）放在同一行。为了完成样式，我们需要一点CSS规则，如列表8.48里显示的。最后的登录表单显示如图8.12。

```css
代码清单 8.48: CSS for the “remember me” checkbox.
# app/assets/stylesheets/custom.css.scss
 .
.
.
/* forms */
.
.
.
.checkbox {
  margin-top: -10px;
  margin-bottom: 10px;
  span {
    margin-left: 20px;
    font-weight: normal;
  }
}

#session_remember_me {
  width: auto;
  margin-left: 0;
}
```
![图8.12：带“记住我”复选框的登陆表单](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/login_form_remember_me.png)

已经编辑了登陆表单，我们现在准备记住用户，假如它们选中复选框，否则忘记他们。令人难以置信的是，因为所有我们在之前节的工作，实现可能被减少到一行。我们通过params哈希为提交的登陆表单现在包含依据复选框的值开始（你可以通过在列表8.47用无效信息提交表单，在页面的调试部分查看值来确认。具体来说，值
```
params[:session][:remember_me]
```
是‘1’，假如框被选中，否则是‘0’。

通过测试相关的params哈希值，我们现在能记住或忘记用户，依据提交的值：
```ruby
if params[:sesssion][:remember_me] == '1'
  remember(user)
else
  forget(user)
end
```
如在旁注8.3里解释过的，这类if-then分支结构能使用三元操作符转换到一行，如下：

```ruby
params[:session][:remember_me] == '1' ? remember(user) : forget(user)
```
添加这个到会话控制器的create方法导致非常好的紧凑代码，显示在列表8.49里。（现在你到了离家列表8.18里的代码的位置了，我们使用三元操作符定义bcrypt
cost变量）
```ruby
代码清单 8.49: Handling the submission of the “remember me” checkbox.
# app/controllers/sessions_controller.rb
 class SessionsController < ApplicationController

  def new
  end

  def create
    user = User.find_by(email: params[:session][:email].downcase)
    if user && user.authenticate(params[:session][:password])
      log_in user
      params[:session][:remember_me] == '1' ? remember(user) : forget(user)
      redirect_to user
    else
      flash.now[:danger] = 'Invalid email/password combination'
      render 'new'
    end
  end

  def destroy
    log_out if logged_in?
    redirect_to root_url
  end
end
```
随着列表8.49里的实现，我们的登录系统完整了，你可以通过在浏览器选中和不选复选框来确认。

旁注8.3 10种人

有一个很老的笑话，世界上10种人，一些懂二进制，一些是不懂二进制的（当然，2的二进制是10）。在这个笑点里，我们也可以说在世界上又10种人，一些是喜欢三元操作符和，一些是不喜欢的，还有就是仍然不知道的。（假如你碰巧是第三类，很快就不是了。）

当你做了许多编程，你很快学到最常用的控制流像这样：

  if boolean?
    do_one_thing
  else
    do_something_else
  end
Ruby，像许多其他语言（包括C/C++, Perl， PHP和Java），允许你用更紧凑的三元操作符替代上面的表达式（之所以称为三元操作符是因为它包含三个部分）：

  boolean? ? do_one_thing : do_something_else
你也能使用三元操作符来替代赋值，如
  if boolean?
    var = foo
  else
    var = bar
  end
变为
  boolean? ? foo : bar
最后，在函数的返回值里使用三元操作符也很方便：

  def foo
    do_stuff
    boolean? ? "bar" : "baz"
  end

因为Ruby隐式地返回函数最后一个表达式的值，这里foo方法依据boolean?是true或false方法"bar"或"baz"。

### 8.4.6 记住测试

尽管我们的“记住我”功能现在工作了，写一些测试来确认它的行为是重要的。原因之一是捕捉实现错误，如上面讨论的。更重要的是，不过，核心用户持久性代码实际上完全没有测试。解决这些问题需要一些技巧，但是结果会试更加有力的测试集。

**测试“记住我”框**

当我在列表8.49里开始实现复选框，与正确的
```ruby
params[:session][:remember_me] == '1' ? remember(user) : forget(user)
```
我实际使用
```ruby
params[:session][:remember_me] ? remember(user) : forget(user)
```
在这个上下文里，params[:session][:remember_me]是‘0’或者‘1’，两个在逻辑环境里都是true，所以表达式的结果总是true，应用程序表现的好像复选框总是选中了。这恰好是错误测试能捕捉的。

因为记住用户要求他们被登陆，我们第一步是定义辅助方法老在测试里让用户登陆。在列表8.20，我们使用post方法和有效的session哈希让用户登陆，但是每次做这个有点啰嗦。为了避免抽欧诺个副，我将写一个log_in_as的辅助方法来为我们登陆。

我们为登陆用户的方法依靠测试的类型。在集成测试里，我们能post到会话路径，如列表8.20，但是在别的测试里（例如控制器和模块测试）这不会工作，我们需要直接操作session方法。结果，log_in_as应该侦测被使用和调整的那种测试。我们能分辨集成测试和其他测试使用Ruby的defind？方法，假如它的参数被定义，返回true，否则返回false。在目前情形，post_via_redirect方法（在列表7.26里见过）仅仅在集成测试里是可用的，所以代码：
```ruby
defined?(post_via_redirect)...
```
将在集成测试里返回true，否则返回false。这暗示定义了一个itegration_test?逻辑方法和写一个if-then语句如下：

```ruby
if integration_test?
  # post到会话路径登陆
else
  # 使用会话登陆
end

```
使用代码填充注释导致了log_in_as辅助方法显示在列表8.50.（这是相当高级的方法，所以你能完全理解它就帮帮哒了。）

```ruby
代码清单 8.50: Adding a log_in_as helper.
# test/test_helper.rb
 ENV['RAILS_ENV'] ||= 'test'
.
.
.
class ActiveSupport::TestCase
  fixtures :all

  # Returns true if a test user is logged in.
  def is_logged_in?
    !session[:user_id].nil?
  end

  # Logs in a test user.
  def log_in_as(user, options = {})
    password    = options[:password]    || 'password'
    remember_me = options[:remember_me] || '1'
    if integration_test?
      post login_path, session: { email:       user.email,
                                  password:    password,
                                  remember_me: remember_me }
    else
      session[:user_id] = user.id
    end
  end

  private

    # Returns true inside an integration test.
    def integration_test?
      defined?(post_via_redirect)
    end
end
```
注意，为了最大程度的灵活，列表8.50里的log_in_as方法接受一个options哈希（如列表7.31），带默认的密码和“记住我”复选框，设置为'password'和‘1’，各自地。具体来说，因为哈希返回nil为不存在的键，代码像
```ruby
remember_me = options[:remember_me] || '1'
```
计算所给的选项，假如存在，否则用默认的。（在旁注8.1里描述的短路求值）。

为了确认“记住我”复选框的行为，我们将写两个测试，一个为提交选中复选框，一个为没选中。使用列表8.50定义的登陆辅助方法是容易的，两种情形显示如
```ruby
log_in_as(@user, remember_me: '1')
```
和
```ruby
log_in_as(@user, remember_me: '0')
```
(因为‘1’是默认的remember_me的值，我们能忽略相应的选中在第一个例子，但是我为了让结构看上去更明显把它包含进来了。）

登陆后，我们能检查是否用户已经被记住了，通过在cookie里查找rememeber_token键。理想的，我们激昂检查cookie的值是和用户的记住口令相等，但是作为目前的设计，对测试来说没有方法读取它：在控制器里的user变量有个记住口令属性，但是（因为remember_token是虚拟的），在测试里@user变量没有。解决这个小的下次作为本章练习（8.6节），但是现在，我们能只测试看见是否相关的cookie是nil或不是。

有更细微的，因为一些原因在测试里cookies方法使用符号作为键不工作，所以
```ruby
cookies[:remember_token]
```
总是nil。幸运的是，cookies键为字符串时工作，所以
```ruby
cookies['remember_token']
```
有我们需要的值。结果测试显示在列表8.51.（回忆列表8.20，users(:michael)参考列表8.19里的fixture）
```ruby
代码清单 8.51: A test of the “remember me” checkbox. 绿色
# test/integration/users_login_test.rb
 require 'test_helper'

class UsersLoginTest < ActionDispatch::IntegrationTest

  def setup
    @user = users(:michael)
  end
  .
  .
  .
  test "login with remembering" do
    log_in_as(@user, remember_me: '1')
    assert_not_nil cookies['remember_token']
  end

  test "login without remembering" do
    log_in_as(@user, remember_me: '0')
    assert_nil cookies['remember_token']
  end
end
```
假设你没有犯我犯的错误，测试应该是绿色的：
```ruby
$ bundle exec rails test
```

** 测试记住分支 **
在8.4.2节，我们手动确认在之前实现的永久性会话工作，但是实际上在current_user
方法里相关的分支现在完全没有测试。我最喜欢的方法处理这种情形是在没有测试的可以的代码块抛出个错误：假如代码没有覆盖，测试将仍然铜鼓；假如覆盖了，结果错误将识别有关的测试。结果在当前情形的显示在列表8.53。

```ruby
代码清单 8.53: Raising an exception in an untested branch. 绿色
# app/helpers/sessions_helper.rb
 module SessionsHelper
  .
  .
  .
  # Returns the user corresponding to the remember token cookie.
  def current_user
    if (user_id = session[:user_id])
      @current_user ||= User.find_by(id: user_id)
    elsif (user_id = cookies.signed[:user_id])
      raise       # The tests still pass, so this branch is currently untested.
      user = User.find_by(id: user_id)
      if user && user.authenticated?(cookies[:remember_token])
        log_in user
        @current_user = user
      end
    end
  end
  .
  .
  .
end

```
在这点，测试是绿色的：
```ruby
代码清单 8.54: 绿色
$ bundle exec rails test
```
这是个问题，当然，因为在列表8.53里的代码崩溃了。而且，手动检查永久性会话是很麻烦的，所以假如我们想要重构current_user方法，（和在第十章一样），测试它是重要的。

因为log_in_as辅助方法定义在列表8.50里自动设置了session[:user_id]，测试“remember”分支的current_user方法在集成测试里是困难的。幸运地，我们能靠测试current_user方法直接在会话辅助方法里测试，它的文件我们不得不创建：
```ruby
$ touch test/helpers/sessions_helper_test.rb
```
测试的顺序是简单的：
1. 使用fixture定义user变量
2. 调用remember方法记住所给的用户
3. 确认current_user和所给用户相同

因为remember方法不设置session[:user_id]，这个过程将测试渴望的“记住”分支。结果在列表8.5里显示。

```ruby
代码清单 8.55: A test for persistent sessions.
# test/helpers/sessions_helper_test.rb
 require 'test_helper'

class SessionsHelperTest < ActionView::TestCase

  def setup
    @user = users(:michael)
    remember(@user)
  end

  test "current_user returns right user when session is nil" do
    assert_equal @user, current_user
    assert is_logged_in?
  end

  test "current_user returns nil when remember digest is wrong" do
    @user.update_attribute(:remember_digest, User.digest(User.new_token))
    assert_nil current_user
  end
end
```
既然我们已经添加了另一个测试，它检查了当前用户是nil，假如用户的记住摘要不能正确地和记住口令匹配，所以测试authenticated?在嵌套的if语句：
```ruby
if user && user.authenticated?(cookies[:remember_token])
```
附带地，在列表8.55我们可能写
```ruby
assert_equal current_user, @user
```
相反，它将一样工作，但是（如在5.6节提过）参数assert_equal的参数被期望，实际上：
```ruby
assert_equal <expected>, <actual>
```
列表8.55里的情形给
```ruby
assert_equal @user, current_user
```
有了列表8.55里的代码，测试如要求的红色：
```
代码清单 8.56: 红色
$ bundle exec rails test TEST=test/helpers/sessions_helper_test.rb
```
我们可以让列表8.55里的测试通过，考移除raise然后恢复到原来的current_user方法，如列表8.57显示。(你也能确认通过移除authenticated?表达式在列表8.57里，第二个测试在列表8.55里是不，确认测试正确的动向。）

```ruby
代码清单 8.57: Removing the raised exception. 绿色
# app/helpers/sessions_helper.rb
 module SessionsHelper
  .
  .
  .
  # Returns the user corresponding to the remember token cookie.
  def current_user
    if (user_id = session[:user_id])
      @current_user ||= User.find_by(id: user_id)
    elsif (user_id = cookies.signed[:user_id])
      user = User.find_by(id: user_id)
      if user && user.authenticated?(cookies[:remember_token])
        log_in user
        @current_user = user
      end
    end
  end
  .
  .
  .
end
```
到这里，测试集应该是绿色的：
```ruby
代码清单 8.58: 绿色
$ bundle exec rails test
```
现在“remember”分支的current_user被测试了，我们能自信地捕捉回归，不用手动检查。

## 8.5 总结
我们覆盖了许多基础，在最后两章，把我们许诺的，但是没有成型的应用程序转为完整的注册和登陆行为的有能力的站点。所有我们吸引的是完成授权功能是限制进入页面依据登陆状态和用户身份。我们将完成这个任务，给用户编辑自己信息的能力，这是第九章的主要目标。

在继续之前，把你的变化合并入主分支：
```ruby
$ bundle exec rails test
$ git add -A
$ git commit -m "Finish log in/log out"
$ git checkout master
$ git merge log-in-log-out
```
然后推送到远程仓库和生产服务器
```ruby
$ bundle exec rails test
$ git push
$ git push heroku
$ heroku run rake db:migrate
```
注意：应用程序将简单地在无效状态，在推送后，但是在迁移结束前。在生产环境有显著流量，改变之前转入维护模式是好想法。：

```ruby
$ heroku maintenance:on
$ git push heroku
$ heroku run rake db:migrate
$ heroku maintenance:off
```

这为显示一个标准的页面错误做准备，在部署和迁移期间。（我们不会再次使用这个，但是起码看看是好的。）要了解更多信息，查看Heroku文档的错误页面。

### 8.5.1 这章我们学到了什么

* Rails可以使用临时和持久的cookie来在页面之间保持状态
* 登陆表单被设计创建用户会话来让用户登陆
* flash.now用来为flash消息渲染页面
* 测试驱动开发当重现测试里的bug时有用。
* 使用session方法，我们能安全地把用户id放置在浏览器，创建临时会话。
* 我们能根据登陆状态改变布局例如链接等
* 集成测试能确认正确的路由，数据库更新，正确的布局变化
* 我们用记住口令和每个用户联系，相应的记住摘要为在永久会话里使用。
* 使用cookies方法，我们创建永久会话通过放置永久记住口令在浏览器cookie
* 登陆状态靠当前用户依据临时会话用户id或者持久会话的唯一记住口令。
* 应用程序署名用户靠删除会话用户id和从浏览器移除永久cookie。
* 三元操作符是压缩简单if-then语句的方法。

## 8.6 练习

1.
在列表8.32，我们定义新的口令和摘要类方法通过显示地给它们User。前缀。这工作很好，因为它们实际调用User.new_token和User.digest，它可能是最清晰的方法定义它们。但是有两个可能在语法上更正确的定义类的方法，有点轻微困惑和一个超级困惑。通过允许测试集，确认列表8.59里的实现（有点困惑）和列表8.60（超级困惑）是正确的。（注意，在列表8.59和8.69里，self是User累，然而在Uesr模型里别的self事业是用户对象实例。这是引起困惑的部分）

2.
如8.4.6节表明的，作为应用程序时当前设计的没有方法进入虚拟的remember_token属性在集成测试里列表8.51.它是可能，不过使用特殊测试方法，叫assigns。在测试里，你能进入实例便于定义在控制器，通过使用assigns用相应的符号。例如，假如create动作定义一个@user变量，我们在测视力使用assigns(:user)读取。现在，会话控制器create动作定义平常（非实例）变量叫user，但是假如我们把它变成实例变量，我们能测试cookies正确地保护用户的记住口令。通过填充列表8.61和8.62消失的部分（用问号？和FILL_IN),完成提高的“记住我”复选框测试。

```ruby
代码清单 8.59: Defining the new token and digest methods using self. 绿色
# app/models/user.rb
 class User < ActiveRecord::Base
  .
  .
  .
  # Returns the hash digest of the given string.
  def self.digest(string)
    cost = ActiveModel::SecurePassword.min_cost ? BCrypt::Engine::MIN_COST :
                                                  BCrypt::Engine.cost
    BCrypt::Password.create(string, cost: cost)
  end

  # Returns a random token.
  def self.new_token
    SecureRandom.urlsafe_base64
  end
  .
  .
  .
end
```

```ruby
class User < ActiveRecord::Base
  .
  .
  .
  class << self
    # Returns the hash digest of the given string.
    def digest(string)
      cost = ActiveModel::SecurePassword.min_cost ? BCrypt::Engine::MIN_COST :
                                                    BCrypt::Engine.cost
      BCrypt::Password.create(string, cost: cost)
    end

    # Returns a random token.
    def new_token
      SecureRandom.urlsafe_base64
    end
  end
  .
```

```ruby
代码清单 8.61: A template for using an instance variable in the create action.
# app/controllers/sessions_controller.rb
 class SessionsController < ApplicationController

  def new
  end

  def create
    ?user = User.find_by(email: params[:session][:email].downcase)
    if ?user && ?user.authenticate(params[:session][:password])
      log_in ?user
      params[:session][:remember_me] == '1' ? remember(?user) : forget(?user)
      redirect_to ?user
    else
      flash.now[:danger] = 'Invalid email/password combination'
      render 'new'
    end
  end

  def destroy
    log_out if logged_in?
    redirect_to root_url
  end
end
```
```ruby
代码清单 8.62: A template for an improved “remember me” test. 绿色
# test/integration/users_login_test.rb
 require 'test_helper'

class UsersLoginTest < ActionDispatch::IntegrationTest

  def setup
    @user = users(:michael)
  end
  .
  .
  .
  test "login with remembering" do
    log_in_as(@user, remember_me: '1')
    assert_equal FILL_IN, assigns(:user).FILL_IN
  end

  test "login without remembering" do
    log_in_as(@user, remember_me: '0')
    assert_nil cookies['remember_token']
  end
  .
  .
  .
end

```


