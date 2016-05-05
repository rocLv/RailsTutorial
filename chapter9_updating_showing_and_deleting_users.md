# 第九章 更新、显示以及删除用户

本章我们通过添加edit、update、index和destroy动作完成用户资源（表7.1）的REST动作。用户可以通过这些操作更新个人信息，这也为我们提供了加强授权模型的机会（第八章里验证代码让这个成为可能）。然后我们将创建所有用户的代码清单（也需要通过验证），这个需求也激发我们介绍生成示例数据和分页。最后，我们将添加删除用户，即把他们从数据库清除的能力。因为我们不允许任何用户有这样危险的能力，所以我们需要在对管理员用户的权限要认真处理，提防别有用心的用户拥有了删除其他用户的能力。

## 9.1 更新用户

编辑用户信息的模式和创建新用户的（第七章）的模式几乎一样。和new动作为新用户渲染新用户视图不一样，我们用edit动作渲染视图来编辑用户信息；和create响应POST请求不同，我们用update动作响应PATCH请求（注3.2）。最大的不同是，任何人都可以注册，然而仅仅当前用户可以更新他们自己的信息。第八章的验证机制允许我们使用before过滤器确保这些得以实现。

让我们从创建updating-users主题分支开始：

```ruby
$ git checkout master
$ git checkout -b updating-users
```

### 9.1.1 编辑表单

我们从编辑表单开始，它的页面原型显示在图9.1。为了实现图9.1，我们需要为用户控制器的edit动作和用户编辑视图添加代码。我们先从edit动作开始，它需要从数据库里获取相关的用户。注意在表7.1力，用户编辑页面的URL是/users/1/edit(假设用户的id是1）。回忆一下用户的id保存在params[:id]变量中，这意味着我们能用代码清单9.1的代码查找用户。
![图9.1：用户个人信息编辑页面的原型](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/edit_user_mockup_bootstrap.png)

```ruby
代码清单 9.1：用户编辑（edit）动作。
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

  def edit
    @user = User.find(params[:id])
  end

  private

    def user_params
      params.require(:user).permit(:name, :email, :password,
                                   :password_confirmation)
    end
end
```

相应的用户编辑视图（你需要手动创建）显示在代码清单9.2里。注意看看它和代码清单7.13的新建用户视图多么相似；大部分代码的重复也提示我们最好把重复的代码放到视图片段里，这个任务留下来作为本章练习（9.6节）。

```ruby
代码清单 9.2：用户编辑视图。
# app/views/users/edit.html.erb
<% provide(:title, "Edit user") %>
<h1>Update your profile</h1>

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

      <%= f.submit "Save changes", class: "btn btn-primary" %>
    <% end %>

    <div class="gravatar_edit">
      <%= gravatar_for @user %>
      <a href="http://gravatar.com/emails" target="_blank">change</a>
    </div>
  </div>
</div>
```

这里我们重复使用了在7.3.3节引入的共享的error_messages视图片段。顺便提一下target="_blank"的使用，它的作用是让浏览器在在新的窗口或者页面打开网页，在链接到第三方网站时是很方便的。

有了代码清单9.1的@user实例变量，edit视图也应该可以正确地被渲染了，如图9.2所示。在图9.2里“姓名”和“Email”的文本框也显示了Rails怎样自动使用@user变量的相关属性来预先填入姓名和Email文本框。

![图9.2：预先填入姓名和email的初始化用户编辑页面](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/edit_page_3rd_edition.png)

看看图9.2的HTML源代码，我们看到了正如我们预料中的一样的表单，如在代码清单9.3里（细节可能略微有点不同）。

```html
代码清单 9.3：代码清单9.2定义的用户个人信息编辑表单HTML，结果如图9.2。
<form accept-charset="UTF-8" action="/users/1" class="edit_user"
      id="edit_user_1" method="post">
  <input name="_method" type="hidden" value="patch" />
  .
  .
  .
</form>
```

注意这里隐藏的文本框
```html
<input name="_method" type="hidden" value="patch" />
```
因为网页浏览器不能原生的发送PATCH请求（如表7.1 REST惯例要求的），Rails通过POST请求和隐藏的input字段伪装成PATCH请求。

有另一个细节需要说明：代码form_for(@user)恰好和代码清单7.13里的一样--所以Rails怎么知道该使用POST请求来创建新用户，还是PATCH来编辑用户呢？答案是通过ApplicationRecord的new_record?逻辑方法可以判断是否是新用户。
```ruby
$ rails console
>> User.new.new_record?
=> true
>> User.first.new_record?
=> false
```

当使用form_for(@user)构建表单时，假如@user.new_record?是true，那么Rails会使用POST方法，否则使用PATCH方法。

作为最后一击，我们把URL添加到网站导航的设置选项里。使用表7.1中的具名路由edit_user_path和代码清单8.36里定义的current_user辅助方法一起很容易实现的这点。

```html
<%= link_to "Settings", edit_user_path(current_user) %>
```
（完整的应用程序代码如代码清单9.4所示）
```ruby
代码清单 9.4：在网站布局中添加“Settings”的URL。
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
              <li><%= link_to "Settings", edit_user_path(current_user) %></li>
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

### 9.1.2 不成功的编辑

在这节，我们将处理不成功的编辑，模仿不成功的注册中相似的想法（7.3节）。我们通过创建update动作开始，它使用update_attributes(6.1.5节）来依据提交的params 哈希更新用户，如代码清单9.5所示。因为对于无效的信息，更新会试图返回false，所以else分支渲染编辑页面。我们之前已经见过这个模式；结构和第一版的create动作极其相似（代码清单7.16）。

```ruby
代码清单 9.5：初始化用户更新动作。
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

  def edit
    @user = User.find(params[:id])
  end

  def update
    @user = User.find(params[:id])
    if @user.update_attributes(user_params)
      # Handle a successful update.
    else
      render 'edit'
    end
  end

  private

    def user_params
      params.require(:user).permit(:name, :email, :password,
                                   :password_confirmation)
    end
end
```

注意在调用update_attributes里user_params的使用，它使用了健壮参数（Strong Parameter）来避免批量赋值引起的安全隐患（如7.3.2节描述的）。

因为存在的用户模型验证和错误信息视图片段在代码清单9.2里，提交无效信息导致有用的错误提示信息（图9.3）。

![图9.3：提交更新表单的错误信息](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/edit_with_invalid_information_3rd_edition.png)

### 9.1.3 测试不成功的编辑

我们在9.1.2节留下了工作的编辑表单。跟随旁注3.3的测试指导，我们现在来写集成测试捕捉回归。我们的第一步是如往常一样生成集成测试：
```ruby
$ rails generate integration_test users_edit
      invoke  test_unit
      create    test/integration/users_edit_test.rb
```
然后我们将写一个简单的不成功编辑的测试，如代码清单9.6所示。在代码清单9.6里的测试通过确认编辑模板在访问编辑页面后被渲染了，然后在提交了无效信息后重新被渲染的行为正确。注意patch方法用来发出PATCH请求，跟随get，post和delete的模式。

```ruby
代码清单 9.6：为不成功的编辑写的测试。绿色
# test/integration/users_edit_test.rb
require 'test_helper'

class UsersEditTest < ActionDispatch::IntegrationTest

  def setup
    @user = users(:michael)
  end

  test "unsuccessful edit" do
    get edit_user+path(@user)
    assert_template 'users/edit'
    patch user_path(@user), user: { name: "",
                                    email: "foo@invalid",
                                    password:              "foo",
                                    password_confirmation: "bar" }
    assert_template 'user/edit'
  end
end
```

在这点，测试集应该仍然是绿色的：
```ruby
代码清单 9.7： 绿色
$ bundle exec rails test
```

### 9.1.4 成功的编辑(用TDD)
是时候让编辑表单工作了。编辑用户头像已经工作了，因为我们已经把图片上传外包给Gravatar了；我们可以通过点击“change”链接，图9.2里的，如在图9.4里显示的。让我们来让剩余的用户编辑功能也能正常工作。

![图9.4：Gravatar图片切割界面，带某帅哥的图片](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/gravatar_cropper.png)

当你越熟悉编写测试，你就越可能发现在编写应用程序代码前写集成测试很有用。在这里，这样的测试有时被称作“验收测试”（acceptance test），因为我们会通过它们（测试）来决定某个功能是否完成。为了看看这种方法怎么工作，我们使用测试驱动开发（TDD）来完成用户个人信息编辑功能。

我们将通过编写和代码清单9.6相似的测试来测试更新用户的行为的正确性。然后我们检查非空的flash信息和重定向到个人信息页面，同时也确认一下数据库里的用户信息已经修改。结果如代码清单9.8所示。注意密码和确认是空的，对于那些只希望修改一下name或email而不想每次都更新密码的用户来说很方便。也注意到@user.reload的使用（第一次是在6.1.5节里见到），我们用它来重新从数据库来加载用户的信息并且确认它们已经成功地更新了。（刚开始时你可能很容易忘记的这种细节，这是为什么验收测试（和普通的TDD)需要一定的经验。）

```ruby
代码清单 9.8：测试成功的编辑。 红色
# test/integration/users_edit_test.rb
require 'test_helper'

class UsersEditTest < ActionDispatch::IntegrationTest

  def setup
    @user = users(:michael)
  end
  .
  .
  .

  test "successful edit" do
    get edit_user_path(@user)
    assert_template 'users/edit'
    name = "Foo Bar"
    email = "foo@bar.com"
    patch user_path(@user), user: { name:  name,
                                    email: email,
                                    password:              "",
                                    password_confirmation: "" }

    assert_not flash.empty?
    assert_redirected_to @user
    @user.reload
    assert_equal name,  @user.name
    assert_equal email, @user.email
  end
end
```

update动作需要代码清单9.8里的测试通过和create动作最后的表单（代码清单8.22）相似，如代码清单9.9所见。

```ruby
代码清单 9.9： 用户更新动作。红色
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  .
  .
  .
  def update
    @user = User.find(params[:id])
    if @user.update_attributes(user_params)
      flash[:success] = "Profile updated"
      redirect_to @user
    else
      render 'edit'
    end
  end
  .
  .
  .
end
```

如在代码清单9.9标题里说明的，测试集仍然是红色的，这是由于代码清单9.8里空的密码和密码确认信息导致密码长度有效性验证失败的结果（代码清单6.39）。为了让测试变绿，我们需要添加空密码时的例外情形。我们可以通过为validates传递allow_nil: true参数实现。如代码清单9.10所示。

```ruby
代码清单 9.10： 允许用空密码更新。绿色
# app/models/user.rb
class User < ActiveRecord::Base
  before_save { self.email = email.downcase }
  validates :name, presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 }
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false }
  has_secure_password
  validates :password, presence: true, length: { minimum: 6 }, allow_nil: true
  .
  .
  .
end
```
如果你担心代码清单9.10可能允许新用户用空密码注册，回想在6.3.3节，has_secure_password在用户对象上已经包含了单独的非空验证。

有了这节的代码，用户信息编辑页面应该可以工作了（图9.5）。你可以通过重新运行测试集来再次确认，它现在应该是绿色：

```ruby
代码清单 9.11： 绿色
$ bundle exec rails test
```
![图9.5：成功编辑的结果页面](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/edit_form_working.png)

## 9.2 权限系统
在Web应用中，用户验证允许我们识别网站的用户，权限系统可以让我们控制他们可以做什么。在第八章建立的验证机制还有个好处就是我们现在已经处于实现权限系统的位置了。

尽管在9.1节完成的编辑和更新动作功能已经完善了，但是它们存在荒唐的安全缺陷：他们允许任何人（甚至没有登陆的用户）进入任意一个动作，任何登陆用户都可以更新别的用户的信息。本节我们要实现一个安全的权限模型，它要求用户登陆，而且阻止他们更新除了自己以外的任何人的信息。

在9.2.1节，我们处理未登陆用户想要访问有权访问的被保护的页面。因为这在正常使用的应用程序中时有发生，一旦发生这种情况，就把把这些用户重定向到登陆页面并为他们提供一些有用的信息，如图9.6的页面原型。另一种情形是有些用户会尝试进入从来不会授权他访问的的页面（例如登陆用户试着访问其他用户的个人信息编辑页面），发生这种情况时就将该用户重定向到根URL（9.2.2节）。

![图9.6：访问被保护页面的结果页面模型](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/login_page_protected_mockup.png)

### 9.2.1 要求登陆用户

为了实现图9.6里显示的重定向行为，我们通过在用户控制器使用“事前过滤器”（before filter）来完成。事前过滤器通过使用before_action命令来让一个特殊的方法在所给定的动作发生之前被调用。为了要求用户登陆，我们定义一个logged_in_user方法，然后使用before_action :logged_in_user唤起该方法，如代码清单9.12所示。
```ruby
代码清单 9.12：在事前过滤器中添加logged_in_user方法。 红色
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  before_action :logged_in_user, only: [:edit, :update]
  .
  .
  .
  private

    def user_params
      params.require(:user).permit(:name, :email, :password,
                                   :password_confirmation)
    end

    # 事前过滤器

    # 确认用户登陆
    def logged_in_user
      unless logged_in?
        flash[:danger] = "Please log in."
        redirect_to login_url
      end
    end
end
```

默认情况下，事前过滤器会应用到控制器里的每个动作，我们也可以通过传递:only哈希选项，限制它仅仅过滤动作:edit和:update。

我们可以在代码清单9.12里通过退出后再次访问用户编辑页面/users/1/edit看到事前过滤器的效果，如图9.7所示。

![图9.7：尝试进入被保护页面后的登陆表单](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/protected_log_in_3rd_edition.png)

如代码清单9.12标题所示，我们的测试集现在是红色的：

```ruby
代码清单 9.13： 红色
$ bundle exec rails test
```
原因是edit和update动作现在都需要用户先登陆，但是在相应的测试里用户还没有登陆。

我们通过在访问edit或update动作前让用户登陆来解决这个问题。使用在8.4.6节开发的log_in_as辅助方法（代码清单8.50）很容易做到这点，如在代码清单9.14里显示的。
```ruby
代码清单 9.14： 让测试用户登陆。 绿色
# test/integration/users_edit_test.rb
 require 'test_helper'

class UsersEditTest < ActionDispatch::IntegrationTest

  def setup
    @user = users(:michael)
  end

  test "unsuccessful edit" do
    log_in_as(@user)
    get edit_user_path(@user)
    .
    .
    .
  end

  test "successful edit" do
    log_in_as(@user)
    get edit_user_path(@user)
    .
    .
    .
  end
end
```

(我们通过把测试登陆放进代码清单9.14里的setup方法来消除重复，但是在9.2.3节里，我们要修改其中一个测试，让测试用户在登陆前访问编辑页面，假如把登陆步骤放在setup中就没法做到这点了。）

在这点，我们的测试集应该是绿色的：
```ruby
代码清单 9.15： 绿色
$ bundle exec rails test
```

即使我们的测试集现在通过了，但是对事事前过滤器器的还没有结束，因为即便我们把安全防护去掉，测试集现在仍然是绿色的。你可把事前过滤器注释掉来确认一下（代码清单9.16）。这很[糟糕](http://catb.org/jargon/html/B/Bad-Thing.html)--我们想让测试集捕捉所有的回归，重大安全漏洞显然是最重要的，所以代码清单9.16里的代码应该是红色。让我们写个测试来捕捉这个问题。

```ruby
代码清单 9.16：为了测试安全模型注释掉事前过滤器。 绿色
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  # before_action :logged_in_user, only: [:edit, :update]
  .
  .
  .
end
```
因为事前过滤器应用在指定的各个动作上，所以我们需要把相应的测试添加到用户控制器。我们计划使用用正确的请求方法访问edit和update动作，然后确认flash被设置，把用户重定向到登陆页面。由表7.1得知，正确的请求方法是用GET和PATCH方法，也就是说在测试中要使用get和patch，如代码清单9.17所示：

```ruby
代码清单 9.17：测试edit和update动作被保护。 红色
# test/controllers/users_controller_test.rb
require 'test_helper'

class UsersControllerTest < ActionController::TestCase

  def setup
    @user = users(:michael)
  end

  test "should get new" do
    get :new
    assert_response :success
  end

  test "should redirect edit when not logged in" do
    get :edit, id: @user
    assert_not flash.empty?
    assert_redirected_to login_url
  end

  test "should redirect update when not logged in" do
    patch :update, id: @user, user: { name: @user.name, email: @user.email }
    assert_not flash.empty?
    assert_redirected_to login_url
  end
end

```
注意get和patch的参数：
···
  get :edit, id: @user
```
和
```
  patch :update, id: @user, user: { name: @user.name, email: @user.email }
```

这里使用了Rails惯例用法id: @user，当在控制器里重定向时Rails会自动使用@user.id。在另一种情形，我们需要提供额外的user哈希，这样路由才能正常工作。（假如你看看第二章的玩具应用生成的用户控制器测试，就会看到上面的代码。）

测试集应该是红色，和我们预料的一样。为了让它变绿，只需要去掉事前过滤器前面的注释符（代码清单9.18）。

```ruby
代码清单 9.18： 撤销注释事前过滤器 绿色
# app/controllers/users_controller.rb
 class UsersController < ApplicationController
  before_action :logged_in_user, only: [:edit, :update]
  .
  .
  .
end
```
有了它，我们的测试集应该是绿色的：

```
代码清单 9.19： 绿色
$ bundle exec rails test
```
现在我们的测试可以立即捕获任何未经授权的用户访问edit动作的行为。

### 9.2.2 要求正确的用户

当然，要求用户登陆还不够；应该仅允许用户编辑他们自己的信息。如在9.2.1节所见，因为测试集缺少一些必要的安全缺陷的测试司空见惯，所以我们将使用测试驱动开发来确认我们的代码正确地实现了安全模型。为了完成这个，我们将把测试添加到用户控器测试来弥补这些。如代码清单9.17所示。

为了确信用户不能编辑别人的信息，我们需要以第二个用户的身份登陆。这意味着我们要在fixture文件里添加第二个用户，如代码清单9.20所示：
```ruby
代码清单 9.20：添加第二个用户到fixture文件。
# test/fixtures/users.yml
 michael:
  name: Michael Example
  email: michael@example.com
  password_digest: <%= User.digest('password') %>

archer:
  name: Sterling Archer
  email: duchess@example.gov
  password_digest: <%= User.digest('password') %>
```

通过使用定义在代码清单8.50里的log_in_as方法，我们可以如代码清单9.21里所示的那样来测试edit和update动作。注意我们想要把用户重定向到根路径而不是登陆路径，因为试着编辑其他用户的用户也可能已经登陆。
```ruby
代码清单 9.21：测试作为错误的用户试着编辑。 红色
# test/controllers/users_controller_test.rb
require 'test_helper'

class UsersControllerTest < ActionController::TestCase

  def setup
    @user       = users(:michael)
    @other_user = users(:archer)
  end

  test "should get new" do
    get :new
    assert_response :success
  end

  test "should redirect edit when not logged in" do
    get :edit, id: @user
    assert_not flash.empty?
    assert_redirected_to login_url
  end

  test "should redirect update when not logged in" do
    patch :update, id: @user, user: { name: @user.name, email: @user.email }
    assert_not flash.empty?
    assert_redirected_to login_url
  end

  test "should redirect edit when logged in as wrong user" do
    log_in_as(@other_user)
    get :edit, id: @user
    assert flash.empty?
    assert_redirected_to root_url
  end

  test "should redirect update when logged in as wrong user" do
    log_in_as(@other_user)
    patch :update, id: @user, user: { name: @user.name, email: @user.email }
    assert flash.empty?
    assert_redirected_to root_url
  end
end
```

为了重定向尝试编辑其他用户信息的用户，我们将添加另一个方法correct_user，并把它添加事前过滤器（代码清单9.22）。注意correct_user事前过滤器定义了@user变量，所以代码清单9.22也显示了我们可以在edit和update动作里删除@user的定义。
```ruby
代码清单 9.22： correct_user事前过滤器用来保护edit/update页面。 绿色
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  before_action :logged_in_user, only: [:edit, :update]
  before_action :correct_user,   only: [:edit, :update]
  .
  .
  .
  def edit
  end

  def update
    if @user.update_attributes(user_params)
      flash[:success] = "Profile updated"
      redirect_to @user
    else
      render 'edit'
    end
  end
  .
  .
  .
  private

    def user_params
      params.require(:user).permit(:name, :email, :password,
                                   :password_confirmation)
    end

    # Before filters

    # Confirms a logged-in user.
    def logged_in_user
      unless logged_in?
        flash[:danger] = "Please log in."
        redirect_to login_url
      end
    end

    # Confirms the correct user.
    def correct_user
      @user = User.find(params[:id])
      redirect_to(root_url) unless @user == current_user
    end
end
```

在这点，我们的测试集应该是绿色的：
```ruby
代码清单 9.23： 绿色
$ bundle exec rails test
```
作为最后的重构，我们将采用通常的惯例，定义current_user?逻辑方法在correct_user事前过滤器中使用，我们把这个方法定义在会话辅助方法（Session helper，代码清单9.24）。我们使用这个方法来代替像下面这样的代码：
```ruby
  unless @user == current_user
```
和（轻微的）更具表达性
```ruby
  unless current_user?(@user)
``````ruby
代码清单 9.24：current_user?方法
# app/helpers/sessions_helper.rb
module SessionsHelper

  # Logs in the given user.
  def log_in(user)
    session[:user_id] = user.id
  end

  # 在持久会话里记住用户。
  def remember(user)
    user.remember
    cookies.permanent.signed[:user_id] = user.id
    cookies.permanent[:remember_token] = user.remember_token
  end

  # 假如所给用户是当前用户，返回true。
  def current_user?(user)
    user == current_user
  end
  .
  .
  .
end
```

用逻辑方法代替直接比较，代码如代码清单9.25所示。
```ruby
代码清单 9.25：最终版的correct_user事前过滤。  绿色
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  before_action :logged_in_user, only: [:edit, :update]
  before_action :correct_user,   only: [:edit, :update]
  .
  .
  .
  def edit
  end

  def update
    if @user.update_attributes(user_params)
      flash[:success] = "Profile updated"
      redirect_to @user
    else
      render 'edit'
    end
  end
  .
  .
  .
  private

    def user_params
      params.require(:user).permit(:name, :email, :password,
                                   :password_confirmation)
    end

    # 事前过滤器

    # 确认用户登陆
    def logged_in_user
      unless logged_in?
        flash[:danger] = "Please log in."
        redirect_to login_url
      end
    end

    # 确认正确的用户
    def correct_user
      @user = User.find(params[:id])
      redirect_to(root_url) unless current_user?(@user)
    end
end
```

### 9.2.3 友好地转发

我们的网站的授权系统已经完善，但是有一点瑕疵：当用户尝试进入受保护页面时他们被重定向到他们自己的个人信息页面，完全无视他们正尝试访问那个页面。也就是说，假如没有登陆的用户试着访问用户编辑页面，登陆后它将被重定向到/users/1而不是/users/1/edit。假如重定向到他们想要的目标地址的话会更友好。

应用程序代码相对有点复杂，但是我们可以写一个极其简单的测试来友好地转发，我们只需通过颠倒登陆和访问编辑页面的顺序即可，如代码清单9.14所示。如在代码清单9.26里所看到的，测试先尝试访问编辑页面，然后登陆，然后检查用户是否被重定向到edit页面，而不是默认的个人信息页面。（代码清单9.26也移除了渲染编辑模板测试，因为那不再是我们想要的行为。）

```ruby
代码清单 9.26：测试友好地转发。红色
# test/integration/users_edit_test.rb
require 'test_helper'

class UsersEditTest < ActionDispatch::IntegrationTest

  def setup
    @user = users(:michael)
  end
  .
  .
  .
  test "successful edit with friendly forwarding" do
    get edit_user_path(@user)
    log_in_as(@user)
    assert_redirected_to edit_user_path(@user)
    name  = "Foo Bar"
    email = "foo@bar.com"
    patch user_path(@user), user: { name:  name,
                                    email: email,
                                    password:              "",
                                    password_confirmation: "" }
    assert_not flash.empty?
    assert_redirected_to @user
    @user.reload
    assert_equal name,  @user.name
    assert_equal email, @user.email
  end
end
```

现在我们已经有了失败的测试，我们准备实现友好地转发。为了把用户导向他们想要的目标地址，我们需要在某些地方存储用户请求的页面地址，然后重定向到那个地址而不是默认地址。我们用一对方法实现这个目标，store_location和redirect_back_or，它们两个都定义在会话辅助方法里（代码清单9.27）。

```ruby
代码清单 9.27：实现友好地转发的代码。
# app/helpers/sessions_helper.rb
module SessionsHelper
  .
  .
  .
  # 重定向到储存的地址（或默认地址）
  def redirect_back_or(default)
    redirect_to(session[:forwarding_url] || default)
    session.delete(:forwarding_url)
  end

  # 储存尝试访问的URL
  def store_location
    session[:forwarding_url] = request.url if request.get?
  end
end
```

这里对转发URL的储存机制是和我们在8.2.1节让用户登陆使用的session工具一样。清单9.27也使用了request对象（通过request.url）来获取请求的页面的URL。

store_location方法在清单9.27里把请求的URL放置到键为:forwarding_url的session变量里，但是仅针对GET请求。这阻止储存转发URL，比如用户在还没登陆时提交表单的情况（这是极端情形，但是也可能发生，假如，例如，用户在提交表单前手动删除了会话cookie。）在这样的情况下，结果重定向将发送GET请求到一个期盼的是POST、PATCH或者DELETE这样的动作，因此引起错误。包含if request.get?语句会阻止这种事情发生。

为了使用store_location，我们需要把它添加到logged_in_user事前过滤器，如清单9.28所示。
```ruby
代码清单 9.28： 把store_location添加到logged_in_user事前过滤器。
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  before_action :logged_in_user, only: [:edit, :update]
  before_action :correct_user,   only: [:edit, :update]
  .
  .
  .
  def edit
  end
  .
  .
  .
  private

    def user_params
      params.require(:user).permit(:name, :email, :password,
                                   :password_confirmation)
    end

    # Before filters

    # Confirms a logged-in user.
    def logged_in_user
      unless logged_in?
        store_location
        flash[:danger] = "Please log in."
        redirect_to login_url
      end
    end

    # Confirms the correct user.
    def correct_user
      @user = User.find(params[:id])
      redirect_to(root_url) unless current_user?(@user)
    end
end
```
为了实现转发自身，如果它存在，我们就通过redirect_back_or方法转发至请求的URL，否则转发至默认的URL，我们添加到会话控制器的create动作来在成功登陆后重定向（清单9。29）。redirect_back_or方法使用或操作符||，通过
```ruby
session[:forwarding_url] || default
```
对session[:forwding_url]求值，除非它是nil，在那种情形时，它的值就是默认的URL。注意清单9.27是小心地移除转发URL；否则后续的登陆企图会转发到保护页面直到用户关掉他们的浏览器。（为这个写测试留下来当作练习（9.6节））也注意即使重定向先显示会话也会被删除；直到显示地return或者方法的结尾重定向不会发生，所以任何在重定向之后的代码仍然会被执行。
```ruby
代码清单 9.29：带友好转发地会话create动作。
# app/controllers/sessions_controller.rb
class SessionsController < ApplicationController
  .
  .
  .
  def create
    user = User.find_by(email: params[:session][:email].downcase)
    if user && user.authenticate(params[:session][:password])
      log_in user
      params[:session][:remember_me] == '1' ? remember(user) : forget(user)
      redirect_back_or user
    else
      flash.now[:danger] = 'Invalid email/password combination'
      render 'new'
    end
  end
  .
  .
  .
end
```
有了它，代码清单9.26的友好地转发集成测试应该可以通过，基本的用户验证和页面保护实现完成了。和往常一样，在进行下一步前确认测试集是绿色的是个好主意：

```ruby
代码清单 9.30： 绿色
$ bundle exec rails test
```

### 9.3 显示所有用户

本节我们将添加倒数第二个用户动作-index动作，它用来显示所有用户，而不是一个用户。通过这节我们将学习怎样为数据库生成示例用户以及怎样分页，这样主页就可以扩展到显示潜在的大量的用户。页面原型的结果--用户、分页链接、和“用户”导航链接--显示在清单9.8。在9.4节，我们将为用户索引添加管理员接口，以便删除用户。

![图9.8：用户主页的页面模型](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_index_mockup_bootstrap.png)

### 9.3.1 用户主页
为了开始用户主页，我们首先实现安全模型。尽管我们将对网站所有访问者开放显示个人用户的show页面，用户index将被限制到登陆用户，以便默认对多少未注册用户能看见有个限制。

为了保护index页面未授权进入，我们将受添加简单的测试来确认index动作被正确的重定向（清单9.31）。
```ruby
代码清单 9.31：测试index动作重定向。 红色
# test/controllers/users_controller_test.rb
require 'test_helper'

class UsersControllerTest < ActionController::TestCase

  def setup
    @user       = users(:michael)
    @other_user = users(:archer)
  end

  test "should redirect index when not logged in" do
    get :index
    assert_redirected_to login_url
  end
  .
  .
  .
end
```
然后我们只需要添加index动作以及在被logged_in_user事前过滤器保护的动作列表里包含它（清单9.32）。
```ruby
代码清单 9.32：index动作要求用户登陆。 绿色
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  before_action :logged_in_user, only: [:index, :edit, :update]
  before_action :correct_user,   only: [:edit, :update]

  def index
  end

  def show
    @user = User.find(params[:id])
  end
  .
  .
  .
end
```

为了显示用户自身，我们需要创建一个包含网站所有用户的变量，然后通过在主页依次渲染。你可能从在玩具应用（清单2.5)里回忆起相应的动作，我们可以使用User.all来从数据库拉取用户，把他们分配给@users实例变量为了在视图里使用，如清单9.33所见。（假如一次立即显示所有用户像是个坏主意，你是对的，我们将在9.3.3节移除这个瑕疵）

```ruby
代码清单 9.33： 用户index动作。
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  before_action :logged_in_user, only: [:index, :edit, :update]
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

为了创建实际的主页，我们要创建视图（你必须手动创建这个文件）来遍历用户，然后把他们打包在li标签里。我们用each方法实现它，在li标签内显示每个用户的Gravatar和姓名（清单9.34）。

```ruby
代码清单 9.34：用户index视图。
# app/views/users/index.html.erb
<% provide(:title, 'All users') %>
<h1>All users</h1>

<ul class="users">
  <% @users.each do |user| %>
    <li>
      <%= gravatar_for user, size: 50 %>
      <%= link_to user.name, user %>
    </li>
  <% end %>
</ul>
```

在清单9.34里的代码使用7.7节代码清单7.31里的结果，它允许我们给Gravatar辅助方法传递除 了默认尺寸之外的选项。假如你没有做这个练习，那么在继续学习前先用代码清单7.31里的内容更新一下Users辅助方法文件。

让我们也添加一点CSS（或SCSS）代码（清单9.35）

```css
代码清单 9.35： 用户主页的CSS
# app/assets/stylesheets/custom.css.scss
.
.
.
/* Users index */

.users {
  list-style: none;
  margin: 0;
  li {
    overflow: auto;
    padding: 10px 0;
    border-bottom: 1px solid $gray-lighter;
  }
}
```

最后，我们在网站的导航栏把这个URL加进去，URL地址为users_path，因此使用表7.1中最后一个具名路由。结果如代码清单9.36显示。
```ruby
代码清单 9.36：为每个用户链接添加URL。
# app/views/layouts/_header.html.erb
<header class="navbar navbar-fixed-top navbar-inverse">
  <div class="container">
    <%= link_to "sample app", root_path, id: "logo" %>
    <nav>
      <ul class="nav navbar-nav navbar-right">
        <li><%= link_to "Home", root_path %></li>
        <li><%= link_to "Help", help_path %></li>
        <% if logged_in? %>
          <li><%= link_to "Users", users_path %></li>
          <li class="dropdown">
            <a href="#" class="dropdown-toggle" data-toggle="dropdown">
              Account <b class="caret"></b>
            </a>
            <ul class="dropdown-menu">
              <li><%= link_to "Profile", current_user %></li>
              <li><%= link_to "Settings", edit_user_path(current_user) %></li>
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

有了它，用户主页功能基本完善，所有测试应该是绿色的：
```ruby
代码清单 9.37： 绿色
$ bundle exec rails test
```

另外如在图9.9里所见，它有点...孤单。让我们急救这个可悲的情形。

![图9.9：只有一个用户的用户主页](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_index_only_one_3rd_edition.png)

### 9.3.2 示例用户们
在这一小节，我们要给我们孤独的用户一些同伴。当然，为了添加足够的用户来创建像样的用户主页，我们可以通过我们浏览器来访问我们网站的注册页面，然后逐个创建新用户。但是更好的解决方案是使用Ruby（或Rake）来为我们创建用户。

首先，我们把Faker gem添加到Gemfile，它允许我们使用半真实的name和email地址（清单9.38）。
```ruby
代码清单 9.38：把Faker gem添加到Gemfile
source 'https://rubygems.org'

gem 'rails',                '4.2.2'
gem 'bcrypt',               '3.1.7'
gem 'faker',                '1.4.2'
.
.
.
```

然后如往常一样安装：
```ruby
$ bundle install
```

接下来，我们添加一个生成示例用户种子库的Rake任务，通常Rails把这个任务添加到db/seeds.rb文件中。结果如清单9.39所示。（在清单9.39里代码有点高级，所以如果看不明白没关系）

```ruby
代码清单 9.39：生成示例用户种子库的Rake任务。
# db/seeds.rb
User.create!(name:  "Example User",
             email: "example@railstutorial.org",
             password:              "foobar",
             password_confirmation: "foobar")

99.times do |n|
  name  = Faker::Name.name
  email = "example-#{n+1}@railstutorial.org"
  password = "password"
  User.create!(name:  name,
               email: email,
               password:              password,
               password_confirmation: password)
end
```
在清单9.39里的代码创建了一个示例用户，复制了我们之前的用户的name和email地址，然后又创建99个其他用户。create!方法和create一样，但是它在用户无效时会抛出例外（6.1.4节）而不是返回false。这有利于我们调试。

有了在清单9.39里的代码，我们可以重置数据库，然后使用rails db:seed来唤醒数据库：
```ruby
$ bundle exec rails db:migrate:reset
$ bundle exec rails db:seed
```
生成种子数据库可能有点慢，在有些系统上要花费几分钟。

在运行完db:seed Rake任务后，我们的应用有了100个示例用户。如在图9.10所见。我已经把前几个示例地址和和Gravatar联系了，以便他们不全是默认的Gravatar图片。（你可能现在不得不重启服务器了。）

![图9.10：有100个用户的主页](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_index_all_3rd_edition.png)

### 9.3.3 分页
我们原始用户不再孤单了，但是现在我们又有了完全相反的问题：我们的用户有太多的同伴了，他们都显示在同一个页面。现在有100个，这已经是合理的大数目了，在真实的网站，可能有成千上万个。解决方法是把这些用户分页显示，以便（例如）每次仅同时显示30个用户。

在Rails中有几种分页方法；我们将使用最简单和最常用的来加速我们的开发，这个gem叫做will_pagenate。为了在程序里使用它，我们需要在Gemfile中包含will_paginate和bootstrap-will_paginate这两个Gem，后面这个Gem用来配置will_paginate使用Bootstrap分页样式。更新后的Gemfile显示在清单9.40。
```ruby
代码清单 9.40： 添加will_paginate Gem到Gemfile。
source 'https://rubygems.org'

gem 'rails',                   '4.2.2'
gem 'bcrypt',                  '3.1.7'``
gem 'faker',                   '1.4.2'
gem 'will_paginate',           '3.0.7'
gem 'bootstrap-will_paginate', '0.0.10'
.
.
.
```

然后运行bundle isntall:
```ruby
$ bundle install
```

你应该重启服务器来确保新的gem正确的加载了。

为了让分页工作，我们还需要添加一些代码到index视图，告诉Rails来把用户分页显示。我们通过在视图里添加特殊的will_paginate方法（清单9.41）开始；我们会看见为什么在用户index代码清单上面和下面都加了这行代码。

```ruby
代码清单 9.41：使用分页的index 视图。
# app/views/users/index.html.erb
<% provide(:title, 'All users') %>
<h1>All users</h1>

<%= will_paginate %>

<ul class="users">
  <% @users.each do |user| %>
    <li>
      <%= gravatar_for user, size: 50 %>
      <%= link_to user.name, user %>
    </li>
  <% end %>
</ul>

<%= will_paginate %>
```

will_paginate方法是有点神奇；在users视图里面，它自动查找@users对象，然后显示进入其他页面的分页链接。在清单9.41里的代码仍然还不能工作，尽管现在@users已经包含了User.all(清单9。33）的结果，然而will_paginate需要我们显示地使用paginate方法给结果分页：

```ruby
$ rails console
>> User.paginate(page: 1)
  User Load (1.5ms)  SELECT "users".* FROM "users" LIMIT 30 OFFSET 0
   (1.7ms)  SELECT COUNT(*) FROM "users"
=> #<ActiveRecord::Relation [#<User id: 1,...
```
注意paginate以哈希作为参数，键为:page和值为请求的页面。User.paginate依据:page参数从数据库里一次拉取一定数量的用户（默认30个）。所以，例如，页面1是用户1-30，那么页面2
是用户31-60，等等。假如page是nil，paginate只会返回第一个页面。

使用paginate方法，我们可以在index动作里使用paginate替换all（清单9.42）。这里page参数来自于params[:page],是will_paginate自动生成的参数。

```ruby
代码清单 9.42：在index动作中为用户分页。
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  before_action :logged_in_user, only: [:index, :edit, :update]
  .
  .
  .
  def index
    @users = User.paginate(page: params[:page])
  end
  .
  .
  .
end
```
用户主页现在应该工作了，显示在图9.11里。（在一些系统，你可能必须重启Rails服务器）因为我们在用户index视图里在代码清单的上部和下部都添加了will_paginate，所以分页链接显示在两个地方。

![图9.11：带分页的用户主页](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_index_pagination_3rd_edition.png)

假如你现在点击2或者next链接，你会得到第二个页面的结果，如图9.12所示。

![图9.12：页面2的用户主页](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_index_page_two_3rd_edition.png)

### 9.3.4 用户主页测试
现在我们的用户主页工作了，我们将为它写个轻量级测试，包含9.3.3节的最小测试。我们的想法是先登陆访问用户主页，然后确认第一页的用户存在，然后确认有分页链接。为了最后这两步工作，我们需要在测试数据库里添加足够多的用户来唤醒分页，例如超过30个。

在清单9.20，我们在fixture里创建了另一个用户，但是30多个用户手动创建的话太多了。幸运地是，我们在使用用户的fixture的password_digest属性时见过，fixture文件支持内嵌Ruby，这意味着我们可以用代码清单9.43里的代码创建30个额外的用户。（清单9.43也创建了几个其他名字的用户，便于将来使用）

```ruby
代码清单 9.43： 为fixture文件添加额外的30个用户。
# test/fixtures/users.yml
michael:
  name: Michael Example
  email: michael@example.com
  password_digest: <%= User.digest('password') %>

archer:
  name: Sterling Archer
  email: duchess@example.gov
  password_digest: <%= User.digest('password') %>

lana:
  name: Lana Kane
  email: hands@example.gov
  password_digest: <%= User.digest('password') %>

mallory:
  name: Mallory Archer
  email: boss@example.gov
  password_digest: <%= User.digest('password') %>

<% 30.times do |n| %>
user_<%= n %>:
  name:  <%= "User #{n}" %>
  email: <%= "user-#{n}@example.com" %>
  password_digest: <%= User.digest('password') %>
<% end %>
```
有了在清单9.43里定义的fixture，我们准备写用户主页的测试。首先我们生成相关的测试：
```ruby
$ rails generate integration_test users_index
      invoke  test_unit
      create    test/integration/users_index_test.rb
```

测试本身需要检查有CSS类名为pagination的div以及确认第一页的用户显示在页面。如代码清单9.44所示。

```ruby
代码清单 9.44：测试包含分页的用户主页。绿色
# test/integration/users_index_test.rb
require 'test_helper'

class UsersIndexTest < ActionDispatch::IntegrationTest

  def setup
    @user = users(:michael)
  end

  test "index including pagination" do
    log_in_as(@user)
    get users_path
    assert_template 'users/index'
    assert_select 'div.pagination'
    User.paginate(page: 1).each do |user|
      assert_select 'a[href=?]', user_path(user), text: user.name
    end
  end
end
```
测试集应该是绿色的
```ruby
代码清单 9.45： 绿色
$ bundle exec rails test
```
### 9.3.5 视图片段重构

分页的用户主页现在已经完成，但是有一个我不能忍受的是：Rails有一些令人难以置信的好工具让我们创建紧凑的视图，在这节里，我们用它们来重构用户主页。因为我们的代码是通过很好的测试的，所以我们可以自信地重构它，确保不会让我们网站崩溃。

我们重构的第一步是用render函数替代代码清单9.41中的用户li标签（清单9.46）
```ruby
代码清单 9.46：index视图的第一次重构。
# app/views/users/index.html.erb
<% provide(:title, 'All users') %>
<h1>All users</h1>

<%= will_paginate %>

<ul class="users">
  <% @users.each do |user| %>
    <%= render user %>
  <% end %>
</ul>

<%= will_paginate %>
```

这里我们调用render，它的参数不是片段视图的的名字而是类User的实例user；在这种上下文中，Rails会自动查找名为_user.html.erb的视图片段，这个视图文件我们必须手动创建（清单9.47）
```ruby
代码清单 9.47：渲染单个用户的视图片段。
# app/views/users/_user.html.erb
<li>
  <%= gravatar_for user, size: 50 %>
  <%= link_to user.name, user %>
</li>
```

这一定是个提升，但是我们可以做的更好：我们能直接在@users变量上调用render（清单9.48）。
```ruby
代码清单 9.48：完美重构后的index视图。 绿色
# app/views/users/index.html.erb
<% provide(:title, 'All users') %>
<h1>All users</h1>

<%= will_paginate %>

<ul class="users">
  <%= render @users %>
</ul>

<%= will_paginate %>
```
这里Rails会推断@users是一列User对象；而且，当调用用户集合时，Rails会自动遍历他们，然后用_user.html.erb来渲染每个视图片段。结果是令人映像深刻的紧凑得代码清单9.48：

在每次重构后，你都应该确认一下测试集仍然是绿色的，改变应用程序代码之后：
```ruby
代码清单 9.49： 绿色
$ bundle exec rails test
```

## 9.4 删除用户
既然我们已经完成了用户主页，就剩下一个REST动作了:destroy。本节我们将添加删除用户的链接，如图9.13的原型图所示，然后定义destroy动作，它是完成删除用户必需的。但是我们先创建管理员用户的类，然后授权管理员删除用户。

![图9.13：带删除链接的用户页面模型](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_index_delete_links_mockup_bootstrap.png)

### 9.4.1 管理员用户

我们将用User模型中的逻辑属性admin来识别拥有特权的管理员，这个逻辑属性会自动在测试中添加admin？逻辑方法来测试用户是否为管理员。数据模型结果显示在图9.14。
![图9.14：用户模型带添加的admin逻辑属性](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_model_admin_3rd_edition.png)

如往常一样，我们用数据迁移添加admin属性，在命令行表明boolean类型：
```ruby
$ rails generate migration add_admin_to_users admin:boolean
```

这个迁移为users表添加admin列，如清单9.50所示。注意我们在清单9.50里给add_column添加了defaul: false参数，这意味着默认用户不是管理员。（如果没有default: false参数，admin默认值为nil，它也是false，所以严格来说不是必须的，不过这样会更加明显，对于Rails和我们代码的读者来说，我们的意图更清晰。）

```ruby
代码清单 9.50：为users表添加逻辑admin属性的数据迁移。
# db/migrate/[timestamp]_add_admin_to_users.rb
class AddAdminToUsers < ActiveRecord::Migration
  def change
    add_column :users, :admin, :boolean, default: false
  end
end
```

接下来，我们如往常一样进行数据迁移：
```ruby
$ bundle exec rails db:migrate
```

如同预料的一样，Rails弄清楚了admin属性的逻辑特性并且自动为我们添加了admin?方法：
```ruby
$ rails console --sandbox
>> user = User.first
>> user.admin?
=> false
>> user.toggle!(:admin)
=> true
>> user.admin?
=> true
```

这里我们使用toggle!方法来切换admin属性。

最后一步，让我们更新一下种子数据库，默认把第一个用户当作管理员（清单9.51）
```ruby
代码清单 9.51：含管理员用户的种子数据代码。
# db/seeds.rb
User.create!(name:  "Example User",
             email: "example@railstutorial.org",
             password:              "foobar",
             password_confirmation: "foobar",
             admin: true)

99.times do |n|
  name  = Faker::Name.name
  email = "example-#{n+1}@railstutorial.org"
  password = "password"
  User.create!(name:  name,
               email: email,
               password:              password,
               password_confirmation: password)
end
```
然后重置数据库：
```ruby
$ bundle exec rails db:migrate:reset
$ bundle exec rails db:seed
```

** 重访“健壮参数”（Strong Parameter） **

你可能已经注意到清单9.51通过在开始的哈希作包含admin: true把第一个用户设为管理员。这强调了把我们的对象暴露到自由的互联网可能面临的危险：恶意用户可以发送恶意的PATCH请求
```ruby
patch /users/17?admin=1
```
这个请求会把用户17设置为管理员，这是个潜在的严重的安全漏洞。

因为具有这种风险，所以我们仅允许更新那些通过网络编辑是安全的属性。如7.3.2节里提到的，我们通过在params哈希上调用require和permit方法使用“健壮参数”（Strong Parameter）来完成这个任务：
```ruby
  def user_params
    require(:user).permit(:name, :email, :password,
                          :password_confirmation)
  end
```

注意admin不在允许的属性列中。这可以阻止任意用户授权他们自己（或其他人）成为我们网站的管理员。因为它的重要性，为任意不可编辑的属性写测试都是好主意。为admin属性写这样的测试是本章的练习（9.6节）。

### 9.4.2 destroy动作

完成User资源的最后一步是添加删除链接和destroy动作。我们通过为每个在主页的用户添加删除链接开始，仅管理员用户才能进入这个页面。结果就是仅仅当用户是管理员时“delete”链接才会显示（清单9.52）。
```ruby
代码清单 9.52：用户删除链接（仅管理员用户能看到）。
# app/views/users/_user.html.erb
<li>
  <%= gravatar_for user, size: 50 %>
  <%= link_to user.name, user %>
  <% if current_user.admin? && !current_user?(user) %>
    | <%= link_to "delete", user, method: :delete,
                                  data: { confirm: "You sure?" } %>
  <% end %>
</li>
```
注意method: :delete参数，为发送必要的DELETE请求的链接做准备。我们也把每个链接包装在if语句里面以便仅管理员用户才能看见他们。如图9.15显示的管理员用户。

网页浏览器不能原生支持发送DELETE请求，所以Rails用Javascript伪装成浏览器原生的DELETE动作。这意味着假如用户不允许使用Javascript那么删除链接就不会工作。假如你必须支持禁用Javascript的浏览器，你可以使用表单POST请求来伪装成DELETE请求，这样即便没有Javascript也可以正常工作。

![图9.15：带删除链接的用户主页](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/index_delete_links_3rd_edition.png)

为了让删除链接工作，我们需要添加destroy动作（表7.1），它首先查找相应的用户，然后用Active Record的destroy方法删除这个用户，最后再重定向到用户主页，如清单9.53所见。因为用户必须先登陆才能删除用户，所以清单9.53也把:destroy添加到logged_in_users事前过滤器中。
```ruby
代码清单 9.53：添加工作的destroy动作。
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  before_action :logged_in_user, only: [:index, :edit, :update, :destroy]
  before_action :correct_user,   only: [:edit, :update]
  .
  .
  .
  def destroy
    User.find(params[:id]).destroy
    flash[:success] = "User deleted"
    redirect_to users_url
  end
  .
  .
  .
end
```

注意destroy动作使用方法链来把find和destroy动作组合成一行：
```ruby
User.find(params[:id]).destroy
```
正如我们构建的仅管理员用户才能通过网站来删除用户，因为仅仅他们才能看见删除链接，但是仍然有个可怕的安全漏洞：任何老练的攻击者仍然可以通过简单发送一个DELETE请求就能直接通过命令行来删除网站的任何用户。为了正确地加强网站安全性，我们也需要对destroy进行读取控制，这样仅管理员才能删除用户。

```ruby
代码清单 9.54：事前过滤器限制管理员才能删除用户。
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  before_action :logged_in_user, only: [:index, :edit, :update, :destroy]
  before_action :correct_user,   only: [:edit, :update]
  before_action :admin_user,     only: :destroy
  .
  .
  .
  private
    .
    .
    .
    # 确认是管理员
    def admin_user
      redirect_to(root_url) unless current_user.admin?
    end
end
```

### 9.4.3 用户删除测试

对于像删除用户这么危险的事情，写一个很好的测试很重要。我们通过添加我们的fixture用户之一作为管理员开始，如清单9.55所示。
```ruby
代码清单 9.55：在fixture中添加一个管理员用户。
# test/fixtures/users.yml
michael:
  name: Michael Example
  email: michael@example.com
  password_digest: <%= User.digest('password') %>
  admin: true

archer:
  name: Sterling Archer
  email: duchess@example.gov
  password_digest: <%= User.digest('password') %>

lana:
  name: Lana Kane
  email: hands@example.gov
  password_digest: <%= User.digest('password') %>

mallory:
  name: Mallory Archer
  email: boss@example.gov
  password_digest: <%= User.digest('password') %>

<% 30.times do |n| %>
user_<%= n %>:
  name:  <%= "User #{n}" %>
  email: <%= "user-#{n}@example.com" %>
  password_digest: <%= User.digest('password') %>
<% end %>
```

以9.2.1节的实践为榜样，我们把动作一级的测试放在User控制器测试文件里。如清单8.28里的用户退出测试一样，我们使用delete来发出DELETE请求，直接发送到destroy动作。我们需要检查两种情形：第一种，没有登陆的用户应该被重定向到登陆页面；第二种，登陆但不是管理员的应该被重定向到主页。结果显示在清单9.56里。

```ruby
代码清单 9.56：管理员读取控制写得动作级测试。 绿色
# test/controllers/users_controller_test.rb
require 'test_helper'

class UsersControllerTest < ActionController::TestCase

  def setup
    @user       = users(:michael)
    @other_user = users(:archer)
  end
  .
  .
  .
  test "should redirect destroy when not logged in" do
    assert_no_difference 'User.count' do
      delete :destroy, id: @user
    end
    assert_redirected_to login_url
  end

  test "should redirect destroy when logged in as a non-admin" do
    log_in_as(@other_user)
    assert_no_difference 'User.count' do
      delete :destroy, id: @user
    end
    assert_redirected_to root_url
  end
end
```

注意清单9.56使用assert_no_difference方法确保用户数量不变（在清单7.21里见过）

在清单9.56里的测试在未授权的（非管理员用户）行为，但是我们也想要检查管理员可以使用删除链接来成功地删除用户。因为删除链接显现在用户主页，所以我们将这些测试添加到清单9.44的用户主页测试。其中仅有的一点技巧是当管理员点击删除链接时确认用户被删除，我们将按照如下完成：
```ruby
assert_difference 'User.count', -1 do
  delete user_path(@other_user)
end
```

这使用了assert_difference方法，第一次在清单7.26见过的，我们在创建用户时使用了它。当发出delete请求到相应的用户路径时通过检查User.count变化了-1来确认用户被删除。

把这些组合在一起给出了清单9.57里的分页和删除测试，那里包含管理员和非管理员的测试。
```ruby
代码清单 9.57：为删除链接和销毁用户写得集成测试。 绿色
# test/integration/users_index_test.rb
require 'test_helper'

class UsersIndexTest < ActionDispatch::IntegrationTest

  def setup
    @admin     = users(:michael)
    @non_admin = users(:archer)
  end

  test "index as admin including pagination and delete links" do
    log_in_as(@admin)
    get users_path
    assert_template 'users/index'
    assert_select 'div.pagination'
    first_page_of_users = User.paginate(page: 1)
    first_page_of_users.each do |user|
      assert_select 'a[href=?]', user_path(user), text: user.name
      unless user == @admin
        assert_select 'a[href=?]', user_path(user), text: 'delete'
      end
    end
    assert_difference 'User.count', -1 do
      delete user_path(@non_admin)
    end
  end

  test "index as non-admin" do
    log_in_as(@non_admin)
    get users_path
    assert_select 'a', text: 'delete', count: 0
  end
end

```
注意清单9.57检查正确的删除链接，包括假如用户碰巧是管理员则跳过测试（由于缺少删除链接清单9.52）。

到这里，我们的删除代码被很好的测试，测试应该是绿色的；
```ruby
代码清单 9.58： 绿色
$ bundle exec rails test
```

## 9.5 总结

我们自从5.4节引入用户控制器已经向前走了很长一段路了。刚开始用户甚至还不能注册；现在用户可以注册、登陆、退出、浏览个人信息以及编辑个人设置，也可以浏览其他用户--有一些用户（管理员）甚至可以删除别的用户。

现在示例程序为任何需要用户验证和授权的的网站形成了稳固的基础。在第十章，我们将添加量过额外的精华：为新注册用户激活账号链接（确认有效的email地址）和帮助忘记密码的用户重置密码。

在我们继续前，确信合并所有的变化进入主分支：
```ruby
$ git add -A
$ git commit -m "Finish user edit, update, index, and destroy actions"
$ git checkout master
$ git merge updating-users
$ git push
```

你也可以部署网站，甚至用示例用户添加到生产数据库（使用pg:reset任务来重置产品数据库）：
```ruby
$ bundle exec rails test
$ git push heroku
$ heroku pg:reset DATABASE
$ heroku run rake db:migrate
$ heroku run rake db:seed
$ heroku restart
```

当然在真实的网站你可能不想要示例数据，但是我为了演示包含进去（图9.16）。顺便提一下，图9.16里的用户顺序可能不同，而且在我们的系统和图9.11也不匹配；这是因为我们没有指明默认的用户排序，所以当前顺序是数据库依赖的。这点不重要，但是排序对于微博来就会显得很重要，我们将在11.14节阐明这个问题。

![图9。16：在生产环境的里的样本用户](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/heroku_sample_users.png)

### 9.5.1 本章我们学到了什么

* 用户能通过发送PATCH请求到update动作，使用编辑表单更新信息
* 使用“健壮参数”（Strong Parameter）使得通过网络更新信息更安全
* 在特殊的控制器动作前，事前过滤器给了运行方法一个标准的方法
* 我们使用事前过滤器实现了授权系统
* 授权测试使用了低级的和和高级的集成测试命令直接提交到特殊的HTTP请求
* 友好的转发在用户登陆后重定向用户到他们想要去页面
* 用户主页显示所有的用户，一次一页
* Rails使用标准文件db/seeds.rb来繁殖数据库，使用rails db:seed用生成样本数据。
* 运行render @users会自动为集合里的每个用户渲染_user.html.erb视图片段
* 逻辑属性admin在用户上的自动生成@user.admin?逻辑方法
* 管理员可以通过点击删除链接发出DELETE请求到User控制器的destroy动作来删除用户
* 我们可以使用内嵌的Ruby在fixture里创建大量的测试用户

## 9.6 练习
1.写一个测试确保友好地转发至第一次访问的URL。对于后续的登陆企图，转向URL应该到默认（例如个人信息页。）提示：在清单9.26里添加测试，通过检查正确的值session[:forwarding_url]。
2. 为了所有的布局链接写一个集成测试，包含登陆和未登陆用户应该显示的正确的链接。暗示：在清单5.25里使用log_in_as辅助方法来添加测试。
3. 通过如清单9.59里所示的发送PATCH请求到update方法，确认admin属性是通过网络不可编辑的。为了确保测试覆盖正确的内容，你第一步应该先把admin添加到user_params的允许参数代码清单，以便初始测试是红色的。
4. 通过重构new.html.erb和edit.html.erb视图来使用清单9.60的视图片段移除重复。注意你必须显式地把f作为局部变量传递给f，如在清单9.61里显示。
```ruby
代码清单 9.59：测试admib属性被禁止。
# test/controllers/users_controller_test.rb
require 'test_helper'

class UsersControllerTest < ActionController::TestCase

  def setup
    @user       = users(:michael)
    @other_user = users(:archer)
  end
  .
  .
  .
  test "should redirect update when logged in as wrong user" do
    log_in_as(@other_user)
    patch :update, id: @user, user: { name: @user.name, email: @user.email }
    assert_redirected_to root_url
  end

  test "should not allow the admin attribute to be edited via the web" do
    log_in_as(@other_user)
    assert_not @other_user.admin?
    patch :update, id: @other_user, user: { password:              FILL_IN,
                                            password_confirmation: FILL_IN,
                                            admin: FILL_IN }
    assert_not @other_user.FILL_IN.admin?
  end
  .
  .
  .
end
```

```ruby
代码清单 9.60：new和edit表单的视图片段。
# app/views/users/_fields.html.erb
<%= render 'shared/error_messages' %>

<%= f.label :name %>
<%= f.text_field :name, class: 'form-control' %>

<%= f.label :email %>
<%= f.email_field :email, class: 'form-control' %>

<%= f.label :password %>
<%= f.password_field :password, class: 'form-control' %>

<%= f.label :password_confirmation, "Confirmation" %>
<%= f.password_field :password_confirmation, class: 'form-control' %>
```

```ruby
代码清单 9.61：使用视图片段的注册视图。
# app/views/users/new.html.erb
<% provide(:title, 'Sign up') %>
<h1>Sign up</h1>

<div class="row">
  <div class="col-md-6 col-md-offset-3">
    <%= form_for(@user) do |f| %>
      <%= render 'fields', f: f %>
      <%= f.submit "Create my account", class: "btn btn-primary" %>
    <% end %>
  </div>
</div>
```

