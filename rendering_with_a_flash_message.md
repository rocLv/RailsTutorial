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

