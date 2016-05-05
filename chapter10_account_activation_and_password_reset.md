# 第十章 账户激活和密码重置

在第九章里，我们完成创建基本的用户资源（填充所有的标准的RESR动作表7.1的），和灵活的验证和授权系统。在这章，我们将通过添加两个非常相关的特性：账户激活（确认新用户的email地址）和重置密码（为那些忘记密码的用户）来完成最后一击。这两个特性重的每个将需要创建新的资源，所以给我们一个看见未来控制器、路欧和数据库迁移的的未来例子。在这个过程，我们也将有机会来学习怎样在Rails里发送邮件，在开发环境和生产环境。最后，这两个特性彼此互补：密码重置需要发送重置链接到用户的email地址，它的有效性通过初始账号激活来确认。

## 10.1 账户激活

目前，新注册用户立即有了完全的进入他们的账户的权限（第七章）。在这节，我们将实现账户机会步骤来确认用户控制他们用来注册的email地址。这也将需要和用户联系的激活口令和摘要，发送给用户带链接的包含口令的email，点击链接激活用户。

我们的策略是处理账户激活与用户登陆相似的方法（8.2节），尤其记住用户（8.4节）。基本的顺序显示如下：

1. 用户开始在“未激活状态”
2. 当用户注册，生成激活口令和相应的激活摘要
3.
把激活摘要保存到数据库，然后给用户发送包含激活口令和用户email地址的链接的email。
4. 当用户点击链接，通过email地址找到用户，然后通过比较激活摘要验证口令。
5.假如用户被验证，把状态从“未激活”改成“激活”

因为和密码和记住口令相似，我们将能服用许多相同的主意为账户激活（和密码重置），包括User.digest和User.new_token方法和修改的user.authenticated?方法。表10.1阐明了模拟（包括密码重置10.2节）。我们将定义生成的版本的authenticated?方法从表10.1在10.1.3节里。

find bystringdigestauthentication
emailpasswordpassword_digestauthenticate(password)
idremember_tokenremember_digestauthenticated?(:remember, token)
emailpasswordpassword_digestauthenticateactivation_tokenactivation_digestauthenticated?(:activation,
token)
emailpasswordpassword_digestauthenticateactivation_tokenactivation_digestauthenticatedreset_tokenreset_digestauthenticated?(:reset,
token)
表10.1：登陆、记住、账户激活和密码重置的模拟

如往常一样，我们将创建主题分支为新特性。如在10.3节所见，账户激活和密码重置包含一些常见的email配置，我们将想要应用两个特性在合并至主分支前。结果，使用蒲长的主题分支很方便。

```ruby
$ git checkout master
$ git checkout -b account-activation-password-reset
```

### 10.1.1 账号激活资源

有了会话（8.1节），我们将模型化账户激活作为资源，即使我们不会和Active
Record模型相联系。相反，我们将包含相关的数据（包括激活口令和激活状态）在用户模型。而且，我们将通过标准的REST
URL来和账户激活交互；因为激活链接将会修改用户的激活状态，我们将计划使用edit动作。这要求一个账户激活控制器，我们能生成如下：
```ruby
$ rails generate controller AccountActivations --no-test-framework
```
注意我们已经包含了跳过测试的标志。这是因为我们不需要控制器测试（更喜欢用集成测试（10.4节）），所以忽略它会方便一点。

激活email需要表单的URL
```ruby
edit_account_activation_url(activation_token, ...)
```
这意味着我们将需要edit动作的命名路由。我们能用显示在青岛10.1里的resources行来安排这个。
```ruby
代码清单 10.1: Adding a resource for account activations.
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
  resources :account_activations, only: [:edit]
end
```

接下来，我们需要唯一的激活口令来激活用户。与密码、记住口令、或者（我们将在10.2节见到）密码重置所有这些一旦被黑客掌握就能控制账户的的东西相比，安全是账户激活最不需考虑的。但是仍然有一些场景比如没有哈希的激活口令会泄露用户。一次，以在8.4节记住口令为例，我们用公开的虚拟口令和数据库里的哈希摘要作为一对。这样我们可以使用
```ruby
user.activation_token
```
来读写激活口令和使用像
```ruby
user.authenticated?(:activation, token)
```
来验证用户。
（这将需要修改authenticated?方法定义在清单8.33里的）我们也加一个逻辑属性activated,这允许我们来测试假如用户使用相同种的自动生成逻辑方法我们在9.4.1节里见过的：
```ruby
if user.activated?
```
最后，经我们不会使用它在本书里，我们将记录激活的时间万一我们在未来会想要它。完整的数据模型显示在途10.1.
![图10.1：添加账户激活属性的用户模型](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_model_account_activation.png)
从图10.1添加数据模型的数据迁移添加所有的三个属性在命令行：
```ruby
$ rails generate migration add_activation_to_users \
> activation_digest:string activated:boolean activated_at:datetime
```
（注意第二行的>是“行连续”符，由shell自动生成的，不要输入），有了admin属性（清单9.50），我们将添加默认的逻辑值false到adtivated属性，如在清单10.2里显示的。
```ruby
代码清单 10.2: A migration for account activation (with added index).
# db/migrate/[timestamp]_add_activation_to_users.rb
 class AddActivationToUsers < ActiveRecord::Migration
  def change
    add_column :users, :activation_digest, :string
    add_column :users, :activated, :boolean, default: false
    add_column :users, :activated_at, :datetime
  end
end
```

我们然后和往常一样应用迁移：
```ruby
$ bundle exec rake db:migrate
```

因为每个新注册用户将要求激活，我们应该分配一个激活口令和摘要给每个用户对象，在创建前。我们看见相似的想法在6.2.5节，在那里我们需要在把email保存到数据库前小写话email地址。在那时，我们使用before_save回叫函数自动在对象保存前调用(清单6.31）。before_save回叫自动调用在对象保存前，包含对象创建和更新，但是这里激活摘要我们仅仅想要回叫拉力当用户创建的适合激活。这需要一个before_create回叫，我们将定义如下：
```ruby
before_create :create_activation_digest
```
这段代码，调用方法参考，安排Rails来查找名为create_activation_digest的方法，然后在创建用户前创建它。（在清单6.31里，我们通过before_save显式的块，但是方法引用技术是更偏好的）因为create_activation_digest方法本身近视眼在用户模型内部，没有必要暴露它给外部用户；如在7.3.2节里，我们见过，完成这个的Ruby之道是使用private关键字：
```ruby
private

  def create_activation_digest
    # Create the token and digest.
  end
```
所有的方法定义在类里的，在private后面的会自动隐藏，如我们在控制台会话里见过的：
```ruby
$ rails console
>> User.first.create_activation_digest
NoMethodError: private method `create_activation_digest' called for #<User>
```

before_create回叫韩式的目的是分配口令和相应的摘要，我们能完成如下：
```ruby
self.activation_token  = User.new_token
self.activation_digest = User.digest(activation_token)
```
这段代码简单的重用了为记住口令使用的口令和摘要方法，如我们通过比较remember方法从清单8.32看到的：
```ruby
# Remembers a user in the database for use in persistent sessions.
def remember
  self.remember_token = User.new_token
  update_attribute(:remember_digest, User.digest(remember_token))
en
```
主要的不同是使用update_attibute在后面的例子。不同的愿意是记住口令和摘要被创建为用户那些已经存在数据库i的，然而before_create回叫函数发生在用户被创建前。回叫的结果，当新用户被定义用User.new（如在用户注册，清单7.17），它将自动还获得avtivation_token和activation_digest属性；因为后者是和数据库里的列相i（图10.1），它将用户保存的时候自动被写。

把上面讨论的综合一下，产生的用户模型显示在清单10.3。正如被虚拟自然的激活口令要求的，我们已经添加了另一个attr_accessor到我们的模型。注意我们已经抓住机会使用email小写的方法参考。

```ruby
代码清单 10.3: Adding account activation code to the User model. 绿色
# app/models/user.rb
 class User < ActiveRecord::Base
  attr_accessor :remember_token, :activation_token
  before_save   :downcase_email
  before_create :create_activation_digest
  validates :name,  presence: true, length: { maximum: 50 }
  .
  .
  .
  private

    # Converts email to all lower-case.
    def downcase_email
      self.email = email.downcase
    end

    # Creates and assigns the activation token and digest.
    def create_activation_digest
      self.activation_token  = User.new_token
      self.activation_digest = User.digest(activation_token)
    end
end
```
在继续前，我们也应该更新我们的种子数据和fixture，以便我们的样例和测试用户出示被激活，如在清单10.4和10.5里的显示。（Time.zone.now方法是内容在Rails的辅助方法，返回当前的时间戳，考虑服务器的时区。）

```ruby
代码清单 10.4: Activating seed users by default.
# db/seeds.rb
 User.create!(name:  "Example User",
             email: "example@railstutorial.org",
             password:              "foobar",
             password_confirmation: "foobar",
             admin:     true,
             activated: true,
             activated_at: Time.zone.now)

99.times do |n|
  name  = Faker::Name.name
  email = "example-#{n+1}@railstutorial.org"
  password = "password"
  User.create!(name:  name,
              email: email,
              password:              password,
              password_confirmation: password,
              activated: true,
              activated_at: Time.zone.now)
end
```
```ruby
代码清单 10.5: Activating fixture users.
# test/fixtures/users.yml
 michael:
  name: Michael Example
  email: michael@example.com
  password_digest: <%= User.digest('password') %>
  admin: true
  activated: true
  activated_at: <%= Time.zone.now %>

archer:
  name: Sterling Archer
  email: duchess@example.gov
  password_digest: <%= User.digest('password') %>
  activated: true
  activated_at: <%= Time.zone.now %>

lana:
  name: Lana Kane
  email: hands@example.gov
  password_digest: <%= User.digest('password') %>
  activated: true
  activated_at: <%= Time.zone.now %>

mallory:
  name: Mallory Archer
  email: boss@example.gov
  password_digest: <%= User.digest('password') %>
  activated: true
  activated_at: <%= Time.zone.now %>

<% 30.times do |n| %>
user_<%= n %>:
  name:  <%= "User #{n}" %>
  email: <%= "user-#{n}@example.com" %>
  password_digest: <%= User.digest('password') %>
  activated: true
  activated_at: <%= Time.zone.now %>
<% end %>
```
为了应用清单10.4里的变化，如往常一样重置数据库来重新繁殖数据：
```ruby
$ bundle exec rake db:migrate:reset
$ bundle exec rake db:seed
```
### 10.1.2 账户激活邮件方法
随着数据模型完成，我们现在准备添加代表发送账户激活email需要的代码。方式是添加用户mailer使用Action
Mailer库，我们在用户控制器里使用来发送带激活链接的email。Mailer的结构更像控制器动作，有email模板定义为试图。我们的任务在这几是定义mailer和视图包含激活口令和与要激活的账户相联系的email地址。

和模型和控制器一样，我们能使用rails generate来生产mailer：
```ruby
$ rails generate mailer UserMailer account_activation password_reset
```
这里我们生产了必要的account_activation方法和我们在10.2节需要的password_reset方法一样。

作为生成mailer的部分，Rails也生成两个视图模板为每个mailer，一个为纯文本email，一个为HTMLemail。为了账户激活mailer方法，他们显示在清单10.6和10.7里。
```ruby
代码清单 10.6: The generated account activation text view.
# app/views/user_mailer/account_activation.text.erb
 UserMailer#account_activation

<%= @greeting %>, find me in app/views/user_mailer/account_activation.text.erb
```
```ruby
代码清单 10.7: The generated account activation HTML view.
# app/views/user_mailer/account_activation.html.erb
 <h1>UserMailer#account_activation</h1>

<p>
  <%= @greeting %>, find me in app/views/user_mailer/account_activation.html.erb
</p>
```
让我们看看生成的mailer来感觉一些它们怎么工作（清单10.8和10.9）。我们在10.8里看见有默认的from地址，在应用程序里所有的mailer，和清单10.9的每个方法也有收件人地址。（清单10.8也使用mailer布局相应带email格式；尽管它不在本书里不重要，HTML和纯文本mailer布局文件能在app/views/layouts里找到。）生成代码页包含一个示例变量（@greeting），在mailer视图里可用，和在控制器里普通的视图可用一样可用的实例变量。
```ruby
代码清单 10.8: The generated application mailer.
# app/mailers/application_mailer.rb
 class ApplicationMailer < ActionMailer::Base
  default from: "from@example.com"
  layout 'mailer'
end
```
```ruby
代码清单 10.9: The generated User mailer.
# app/mailers/user_mailer.rb
 class UserMailer < ApplicationMailer

  # Subject can be set in your I18n file at config/locales/en.yml
  # with the following lookup:
  #
  #   en.user_mailer.account_activation.subject
  #
  def account_activation
    @greeting = "Hi"

    mail to: "to@example.org"
  end

  # Subject can be set in your I18n file at config/locales/en.yml
  # with the following lookup:
  #
  #   en.user_mailer.password_reset.subject
  #
  def password_reset
    @greeting = "Hi"

    mail to: "to@example.org"
  end
end
```
为了创建工作的激活email，我们将首先自定义生成如在清单10.10显示的模板。接下来，我们将创建实例变量包含用户（为了在视图里使用），然后邮递结果到user.email(清单10.11）。如在清单10.11里所见，mail方法也有个subject键，它的值是邮件的主题。
```ruby
代码清单 10.10: The application mailer with a new default from address.
# app/mailers/application_mailer.rb
 class ApplicationMailer < ActionMailer::Base
  default from: "noreply@example.com"
  layout 'mailer'
end
```
```ruby
代码清单 10.11: Mailing the account activation link.
# app/mailers/user_mailer.rb
 class UserMailer < ApplicationMailer

  def account_activation(user)
    @user = user
    mail to: user.email, subject: "Account activation"
  end

  def password_reset
    @greeting = "Hi"

    mail to: "to@example.org"
  end
end
```
作为普通的视图，我们可以使用内嵌Ruby来自定义模板视图，在这里，通过姓名问候用户，然后包含一个自定义激活链接。我们的计划是查找用户依靠email地址，然后验证激活口令，所以链接需要包含email和口令。因为我们正模型化激活使用Account
Activation资源，口令本身能显示作为定义得清单10.1里的命名路由：
```ruby
edit_account_activation_url(@user.activation_token, ...)
```
回忆
```ruby
edit_user_url(user)
```
输出了表单的URL
```ruby
http://www.example.com/users/1/edit
```
相应的账户激活链接的URL基础看起来像这个：
```ruby
http://www.example.com/account_activations/q5lt38hQDc_959PVoo6b7A/edit
```
这里q5lt38hQDc_959PVoo6b7A是URL安全的base64字符串，通过new_token方法生成（清单8.31），它和在users/1/edit里的用户id扮演同样的角色。具体来说，在Activation控制器的edit动作I里，口令在在params哈希里如params[:id]一样可用。

为了也包含email，我们需要使用查询参数，在URL里显示作为键-值对，位于问号后：
```ruby
# account_activations/q5lt38hQDc_959PVoo6b7A/edit?email=foo%40example.com
```
注意‘@’在email地址里的显示为%40，例如，它是转义字符，为了保证有效的URL。在Rails里设置查询参数的方法是在命名路由里包含哈希：
```ruby
edit_account_activation_url(@user.activation_token, email: @user.email)
```
当这样使用命名路由来定义查询参数，Rails自动转义任何特殊字符。email地址也会在控制器里自动复原，通过params[:email]可用。

有了@user实例变量定义在清单10.11里，我们能创建必要的链接使用命名的编辑路由和内嵌Ruby，如显示在清单10.12和10.13里。注意HTML模板在清单10.13里的使用了link_to方法来构建有效的链接。
```ruby
代码清单 10.12: The account activation text view.
# app/views/user_mailer/account_activation.text.erb
 Hi <%= @user.name %>,

Welcome to the Sample App! Click on the link below to activate your account:

<%= edit_account_activation_url(@user.activation_token, email: @user.email) %>
```

```ruby
代码清单 10.13: The account activation HTML view.
# app/views/user_mailer/account_activation.html.erb
 <h1>Sample App</h1>

<p>Hi <%= @user.name %>,</p>

<p>
Welcome to the Sample App! Click on the link below to activate your account:
</p>

<%= link_to "Activate", edit_account_activation_url(@user.activation_token,
                                                    email: @user.email) %>
```

为了看见定义在清单10.12和10.13里的模板结果，我们能使用email预览，它是Rails暴露的一些特殊的URL，让我们看见我们的email信息看起来怎么样。首先，我们需要添加一些配置到我们的>应用程序的环境，如显示在清单10.14里。
```ruby
代码清单 10.14: Email settings in development.
# config/environments/development.rb
 Rails.application.configure do
  .
  .
  .
  config.action_mailer.raise_delivery_errors = true
  config.action_mailer.delivery_method = :test
  host = 'example.com'
  config.action_mailer.default_url_options = { host: host }
  .
  .
  .
end
```
清单10.14使用了主机名为'example.com',但是你赢使用你开发环境的真实主机。例如，在我们的系统下面两个中的一个（依赖是否我正使用云IDE或本地服务器）：
```ruby
host = 'rails-tutorial-c9-mhartl.c9.io'     # Cloud IDE
```
```ruby
host = 'localhost:3000'                     # Local server

```
重启开发服务器后激活在清单10.14里的配置文件，我们接下来学更新用户mailer预览文件，我们在10.12节列自动生成的，如显示在清单10.15.
```ruby
代码清单 10.15: The generated User mailer previews.
# test/mailers/previews/user_mailer_preview.rb
 # Preview all emails at http://localhost:3000/rails/mailers/user_mailer
class UserMailerPreview < ActionMailer::Preview

  # Preview this email at
  # http://localhost:3000/rails/mailers/user_mailer/account_activation
  def account_activation
    UserMailer.account_activation
  end

  # Preview this email at
  # http://localhost:3000/rails/mailers/user_mailer/password_reset
  def password_reset
    UserMailer.password_reset
  end

end
```

因为在清单10.11里定义的account_activation方法要求有效的用户对象作为参数，在清单10.15里的代码不会如写的那样工作。为了解决它，我们定义user变量等于开发数据库里的第一个用户，然后作为UserMailer.accounta_activation(清单10.16）的参数。注意清单10.16也给user.activation_token赋值，这是必要的，因为在清单10.12和10.13里的激活模板需要账户激活口令。（因为activation_token是虚拟属性（10.1.1节），从数据库里的用户没有）

```ruby
代码清单 10.16: A working preview method for account activation.
# test/mailers/previews/user_mailer_preview.rb
 # Preview all emails at http://localhost:3000/rails/mailers/user_mailer
class UserMailerPreview < ActionMailer::Preview

  # Preview this email at
  # http://localhost:3000/rails/mailers/user_mailer/account_activation
  def account_activation
    user = User.first
    user.activation_token = User.new_token
    UserMailer.account_activation(user)
  end

  # Preview this email at
  # http://localhost:3000/rails/mailers/user_mailer/password_reset
  def password_reset
    UserMailer.password_reset
  end
end
```
有了清单10.16里的预览代码，我们能访问建议的URL来预览账户激活电子邮件。（假如你正使用云IDE，你应该用相应的基础URL替换localhost:3000）结果HTML和文本邮件显示在图10.2和10.3里。

![图10.2：预览HTML版的账户激活电子邮件](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/account_activation_html_preview.png)

![图10.3：纯文本版的账户激活电子邮件](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/account_activation_text_preview.png)

作为最后一步，我们将写几个测试来确保显示在电子邮件里的预览的结果。这不是和它听起来一样难，因为Rails为我们生成了有用的示例测试（清单10.17）。
```ruby
代码清单 10.17: The User mailer test generated by Rails.
# test/mailers/user_mailer_test.rb
 require 'test_helper'

class UserMailerTest < ActionMailer::TestCase

  test "account_activation" do
    mail = UserMailer.account_activation
    assert_equal "Account activation", mail.subject
    assert_equal ["to@example.org"], mail.to
    assert_equal ["from@example.com"], mail.from
    assert_match "Hi", mail.body.encoded
  end

  test "password_reset" do
    mail = UserMailer.password_reset
    assert_equal "Password reset", mail.subject
    assert_equal ["to@example.org"], mail.to
    assert_equal ["from@example.com"], mail.from
    assert_match "Hi", mail.body.encoded
  end
end
```

清单10.17里的测试使用了强大的assert_match方法，它既可以使用字符串，也可以使用正则表达式：
```ruby
assert_match 'foo', 'foobar'   # true
assert_match 'baz', 'foobar'   # false
assert_match '\w+', 'foobar'   # true
assert_match '\w+', '$#!*+@'   # false
```
在清单10.18里的测试使用assert_match来检查姓名，激活口令，和转义过的email，显示在email主体。这些最后的，注意
```ruby
CGI::escape(user.email)
```

```ruby
代码清单 10.18: A test of the current email implementation. 红色
# test/mailers/user_mailer_test.rb
 require 'test_helper'

class UserMailerTest < ActionMailer::TestCase

  test "account_activation" do
    user = users(:michael)
    user.activation_token = User.new_token
    mail = UserMailer.account_activation(user)
    assert_equal "Account activation", mail.subject
    assert_equal [user.email], mail.to
    assert_equal ["noreply@example.com"], mail.from
    assert_match user.name,               mail.body.encoded
    assert_match user.activation_token,   mail.body.encoded
    assert_match CGI::escape(user.email), mail.body.encoded
  end
end
```
注意到清单10.18会添加激活口令到fixture用户，否则将是空白的。

为了让测试通过，我们不得不配置我们的测试文件用正确的主机域名，如在清单10.19里显示的。
```ruby
代码清单 10.19: Setting the test domain host.
# config/environments/test.rb
 Rails.application.configure do
  .
  .
  .
  config.action_mailer.delivery_method = :test
  config.action_mailer.default_url_options = { host: 'example.com' }
  .
  .
  .
end
```
有了上面的代码，mailer测试应该是绿色的：
```ruby
代码清单 10.20: 绿色
$ bundle exec rails test:mailers
```
为了在我们的应用程序里使用mailer，我们只需要添加几行到用户注册需要的create动作，如清单10.21显示。注意清单10.21已经改变了重定向行为在注册。以前，我们重定向到用户的个人信息也（7.4节），但是现在不合理了，因为我们要求账户激活。想法，我们现在重定向到根URL。

```ruby
代码清单 10.21: Adding account activation to user signup. 红色
# app/controllers/users_controller.rb
 class UsersController < ApplicationController
  .
  .
  .
  def create
    @user = User.new(user_params)
    if @user.save
      UserMailer.account_activation(@user).deliver_now
      flash[:info] = "Please check your email to activate your account."
      redirect_to root_url
    else
      render 'new'
    end
  end
  .
  .
  .
end
```
因为清单10.21重定向到根URL而不是个人信息也，不是像之前一样登陆。测试集现在是红色，及时即使应用程序是按照设计的工作。我们将临时通过注释掉失败的行来解决，如显示在清单10.22里。我们将在10.1.4节取消那些行的注释和写为账户激活的测试。
```ruby
代码清单 10.22: Temporarily commenting out failing tests. 绿色
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
    assert_select 'div#error_explanation'
    assert_select 'div.field_with_errors'
  end

  test "valid signup information" do
    get signup_path
    assert_difference 'User.count', 1 do
      post_via_redirect users_path, user: { name:  "Example User",
                                            email: "user@example.com",
                                            password:              "password",
                                            password_confirmation: "password" }
    end
    # assert_template 'users/show'
    # assert is_logged_in?
  end
end
```
假如你现在正尝试作为一个新用户注册，你应该被重定向到图10.4显示的页面，email像这个显示在清单10.23应该被生成。注意你不会在开发环境里收到真实的email，但是它会在你服务器日志里显示。（你可能不得能滚动一下然后看见它）10.3节讨论怎样在生产环境发送email。

```ruby
代码清单 10.23: A sample account activation email from the server log.
Sent mail to michael@michaelhartl.com (931.6ms)
Date: Wed, 03 Sep 2014 19:47:18 +0000
From: noreply@example.com
To: michael@michaelhartl.com
Message-ID:
<540770474e16_61d3fd1914f4cd0300a0@mhartl-rails-tutorial-953753.mail>
Subject: Account activation
Mime-Version: 1.0
Content-Type: multipart/alternative;
 boundary="--==_mimepart_5407704656b50_61d3fd1914f4cd02996a";
 charset=UTF-8
Content-Transfer-Encoding: 7bit


----==_mimepart_5407704656b50_61d3fd1914f4cd02996a
Content-Type: text/plain;
 charset=UTF-8
Content-Transfer-Encoding: 7bit

Hi Michael Hartl,

Welcome to the Sample App! Click on the link below to activate your account:

http://rails-tutorial-c9-mhartl.c9.io/account_activations/
# fFb_F94mgQtmlSvRFGsITw/edit?email=michael%40michaelhartl.com
----==_mimepart_5407704656b50_61d3fd1914f4cd02996a
Content-Type: text/html;
 charset=UTF-8
Content-Transfer-Encoding: 7bit

<h1>Sample App</h1>

<p>Hi Michael Hartl,</p>

<p>
Welcome to the Sample App! Click on the link below to activate your account:
</p>

<a href="http://rails-tutorial-c9-mhartl.c9.io/account_activations/
# fFb_F94mgQtmlSvRFGsITw/edit?email=michael%40michaelhartl.com">Activate</a>
----==_mimepart_5407704656b50_61d3fd1914f4cd02996a--
```
![图10.4：注册后带激活信息的主页](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/redirected_not_activated.png)

### 10.1.3 激活账户

现在我们有正确地生成email在清单10.23里，我们需要在Account Activation控制器edit动作里来激活用户。回忆在10.12节的讨论激活口令和email是作为params[:id]和params[:email]是可用的，各自。以密码模型和记住口令为例（清单8.36），我们计划查找和验证用户用像这样的代码：
```ruby
user = User.find_by(email: params[:email])
if user && user.authenticated?(:activation, params[:id])
```
(和我们一会就会看见的的一样，在表达式里将有个额外的逻辑上面。看看你能不能猜到会是什么。）

上面的代码使用authenticated?方法来测试是否账户激活摘要和所给的口令匹配，但是目前，这不会工作，因为方法对记住口令是专有化的（清单8.33）：
```ruby
# Returns true if the given token matches the digest.
def authenticated?(remember_token)
  return false if remember_digest.nil?
  BCrypt::Password.new(remember_digest).is_password?(remember_token)
end
```
这里remember_digests是User模型的属性，在模型里我们可以重写它为：
```ruby
self.remember_digest
```
出于某种原因，我们想要能创建这个变量，所以我们能调用
```ruby
self.activation_token
```
通过传递合适的参数到authenticated?

解决需要我们第一个元编程的例子，对于写程序的程序来说是必要的。（元编程是Ruby最强大的套件之一，和许多Rails的“魔法”特性正是因为使用了Ruby的元编程）这个例子的关键是强大的send方法，让我们调用方法用我们选择的名字通过“发送信息”到一个给定对象。例如，在这个控制台会话我们使用send在原生的Ruby对象来查找数组的长度：
```ruby
$ rails console
>> a = [1, 2, 3]
>> a.length
=> 3
>> a.send(:length)
=> 3
>> a.send('length')
=> 3
```
这里我们看见传递符号:length或字符串'length'到send时和调用length方法在所给对象是等价的。作为第二个例子，我们读取数据库里第一个用户的activation_digest：
```ruby
>> user = User.first
>> user.activation_digest
=> "$2a$10$4e6TFzEJAVNyjLv8Q5u22ensMt28qEkx0roaZvtRcp6UZKRM6N9Ae"
>> user.send(:activation_digest)
=> "$2a$10$4e6TFzEJAVNyjLv8Q5u22ensMt28qEkx0roaZvtRcp6UZKRM6N9Ae"
>> user.send('activation_digest')
=> "$2a$10$4e6TFzEJAVNyjLv8Q5u22ensMt28qEkx0roaZvtRcp6UZKRM6N9Ae"
>> attribute = :activation
>> user.send("#{attribute}_digest")
=> "$2a$10$4e6TFzEJAVNyjLv8Q5u22ensMt28qEkx0roaZvtRcp6UZKRM6N9Ae"
```
注意在最后的例子，我们已经定义了attribute变量和符号:activation相等，使用字符串插值来组成正确的参数到send。这也将对字符串'activation'工作，但是使用符号是更符合惯例，和在另一个情形
```ruby
"#{attribute}_digest"
```
变成
```ruby
"activation_digest"
```
一旦字符串被插值。（我们在7.4.2节里见过符号怎么作为字符串插值）

依据对send的讨论，我们能重写当前的authenticated?方法如下：
```ruby
  def authenticated?(remember_token)
    digest = self.send('remember_digest')
    return false if digest.nil?
    BCrypt::Password.new(digest).is_password?(remember_token)
  end
```
有了这个模板,我们能通过田间函数参数带摘要的名字然后生成一般化的方法，然后如上面一样使用字符串插值：
```ruby
def authenticaated?(attribute, token)
  digest = self.send("#{attribute}_digest")
  return false if digest.nil?
  BCrypt::Password.new(digest).is_password?(token)
end
```
(这里我们已经重命名了第二个参数为token来强调它现在一般化了。）
因为我们是在用户模型里面，我们也能忽略self，生成最地道的正确版本：
```ruby
def authenticated?(attribute, token)
  digest = send("#{attribute}_digest"
  return false if digest.nil?
  BCrypt::Password.new(digest).is_password?(token)
end
```
我们现在可以通过像这样：
```ruby
user.authenticated?(:remember, remember_token)
```
来重现之前authenticated?的行为。
应用这个对User模型的的讨论，生成了一般化的authenticated?方法，如清单10.24所示。
```ruby
代码清单 10.24: A generalized authenticated? method. 红色
# app/models/user.rb
 class User < ActiveRecord::Base
  .
  .
  .
  # Returns true if the given token matches the digest.
  def authenticated?(attribute, token)
    digest = send("#{attribute}_digest")
    return false if digest.nil?
    BCrypt::Password.new(digest).is_password?(token)
  end
  .
  .
  .
end
```
清单10.24的标题表明了红色的测试集：
```ruby
代码清单 10.25: 红色
$ bundle exec rails test
```
失败的原因是因为current_user方法（清单8.36）和对nil摘要的测试（清单8.43）两个都使用旧版的authenticated?，期望一个参数，而不是两个。为了解决这个，我们简单更新两种情况使用一般化的方法，如显示在清单10.26和清单10.27里。
```ruby
代码清单 10.26: Using the generalized authenticated? method in current_user.
# app/helpers/sessions_helper.rb
 module SessionsHelper
  .
  .
  .
  # Returns the current logged-in user (if any).
  def current_user
    if (user_id = session[:user_id])
      @current_user ||= User.find_by(id: user_id)
    elsif (user_id = cookies.signed[:user_id])
      user = User.find_by(id: user_id)
      if user && user.authenticated?(:remember, cookies[:remember_token])
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
```ruby
代码清单 10.27: Using the generalized authenticated? method in the User test.
绿色
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
    assert_not @user.authenticated?(:remember, '')
  end
end
```
在这个点，测试应该是绿色的：
```ruby
代码清单 10.28: 绿色
$ bundle exec rails test
```
如上重构代码如果没有完整的测试集肯定非常容易出错，这是为什么我们在8.4.2和8.4.6节不嫌麻烦写好的测试。

有了在清单10.24里的authenticated?方法如清单10.24里的，我们现在准备写edit动作，验证params哈希里的email地址相对应的用户。我们的有效性测试将看起来像这：
```ruby
if user && !user.activated? && user.authenticated?(:activation, params[:id])
```
注意!user.activated?的出现，这是在上面间接提到的额外的逻辑判断。这可以阻止我们的代码对已经激活的用户再次激活，这是重要的，因为我们将确认用户登陆，我们不想允许掌握了激活链接的黑客登陆。

假如用户被验证，根据上面的逻辑判断，我们需要激活用户和更新activated_at时间戳。

```ruby
user.update_attribute(:activated,    true)
user.update_attribute(:activated_at, Time.zone.now)
```
这导出清单10.29显示的edit动作。也注意清单10.29处理无效激活口令的情形；这几乎不会发生，但是在这种情形重定向到根URL是足够容易的。
```ruby
代码清单 10.29: An edit action to activate accounts.
# app/controllers/account_activations_controller.rb
 class AccountActivationsController < ApplicationController

  def edit
    user = User.find_by(email: params[:email])
    if user && !user.activated? && user.authenticated?(:activation, params[:id])
      user.update_attribute(:activated,    true)
      user.update_attribute(:activated_at, Time.zone.now)
      log_in user
      flash[:success] = "Account activated!"
      redirect_to user
    else
      flash[:danger] = "Invalid activation link"
      redirect_to root_url
    end
  end
end
```
有了清单10.29里的代码，你现在应该能从清单10.23粘贴URL来激活相关的用户。例如，在我的系统我访问了URL
```ruby
http://rails-tutorial-c9-mhartl.c9.io/account_activations/
# fFb_F94mgQtmlSvRFGsITw/edit?email=michael%40michaelhartl.com
```
然后结果如图10.5所示。

![图10.5:成功激活后的个人信息页面](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/activated_user.png)

当然，当前用户激活没有真正地做什么。因为我们没有改变用户怎样登陆。为了让用户激活意味些什么，我们需要只允许激活用户登陆。如清单10.30所示，做这个的方法是假如user.activated?是true，用户就像往常一样登陆；否则，重定向到代warning信息的根URL（图10.6）。

```ruby
代码清单 10.30: Preventing unactivated users from logging in.
# app/controllers/sessions_controller.rb
 class SessionsController < ApplicationController

  def new
  end

  def create
    user = User.find_by(email: params[:session][:email].downcase)
    if user && user.authenticate(params[:session][:password])
      if user.activated?
        log_in user
        params[:session][:remember_me] == '1' ? remember(user) : forget(user)
        redirect_back_or user
      else
        message  = "Account not activated. "
        message += "Check your email for the activation link."
        flash[:warning] = message
        redirect_to root_url
      end
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

![图10.6：仍然没有激活的用户的警告信息](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/not_activated_warning.png)

有了那个，有一点不同，基本的用户激活功能结束了。（那点是不显示未激活用户，这作为本章的练习（10.5节）。在10.1.4节，我们将通过添加一些测试然后重构来结束这个过程。

### 10.1.4 激活测试和重构

在这节，我们将为账户激活添加集成测试。因为我们已经有了为带有效信息注册的测试，我们将添加步骤来测试在7.4.4节（清单7.26）开发的测试。有很少基本，但是假如你能跟着清单10.31，它们大部分是直白的。

```ruby
代码清单 10.31: Adding account activation to the user signup test. 绿色
# test/integration/users_signup_test.rb
 require 'test_helper'

class UsersSignupTest < ActionDispatch::IntegrationTest

  def setup
    ActionMailer::Base.deliveries.clear
  end

  test "invalid signup information" do
    get signup_path
    assert_no_difference 'User.count' do
      post users_path, user: { name:  "",
                               email: "user@invalid",
                               password:              "foo",
                               password_confirmation: "bar" }
    end
    assert_template 'users/new'
    assert_select 'div#error_explanation'
    assert_select 'div.field_with_errors'
  end

  test "valid signup information with account activation" do
    get signup_path
    assert_difference 'User.count', 1 do
      post users_path, user: { name:  "Example User",
                               email: "user@example.com",
                               password:              "password",
                               password_confirmation: "password" }
    end
    assert_equal 1, ActionMailer::Base.deliveries.size
    user = assigns(:user)
    assert_not user.activated?
    # Try to log in before activation.
    log_in_as(user)
    assert_not is_logged_in?
    # Invalid activation token
    get edit_account_activation_path("invalid token")
    assert_not is_logged_in?
    # Valid token, wrong email
    get edit_account_activation_path(user.activation_token, email: 'wrong')
    assert_not is_logged_in?
    # Valid activation token
    get edit_account_activation_path(user.activation_token, email: user.email)
    assert user.reload.activated?
    follow_redirect!
    assert_template 'users/show'
    assert is_logged_in?
  end
end
```
在清单10.31里有许多代码，但是仅仅有一行是新加的
```ruby
assert_equal 1, ActionMailer::Base.deliveries.size
```
代码确认恰好有1个信息被发送。因为deliveries数组是全局的，我们不得不在setup方法里重置它，防止有别的测试发送邮件打碎我们的代码（和在10.2.5节里的情形一样）。清单10.31是在本书中第一次使用assigns方法；如在第八章练习（8.6节）解释的，assigns让我们读取在相应的动作里的实例变量。例如，User控制器的create动作定义了@user变量（清单10.21），所以我们能在测视力使用assigns(:user)来读写它。最后，注意在清单10.31里恢复了我们在清单10.22里注释掉的代码。

在这个点，测试集应该是绿色的：
```ruby
代码清单 10.32: 绿色
$ bundle exec rails test
```
有了清单10.31里的测试，我们准备通过移动一些用户的操作从控制器到模型里来轻轻重构一下。具体来说，我们将创建activate方法来更新用户的激活属性和send_activation_email来发送激活邮件。额外的方法显示在清单10.33里，重构的应用程序代码些事在清单10.34和10.35里。
```ruby
代码清单 10.33: Adding user activation methods to the User model.
# app/models/user.rb
 class User < ActiveRecord::Base
  .
  .
  .
  # Activates an account.
  def activate
    update_attribute(:activated,    true)
    update_attribute(:activated_at, Time.zone.now)
  end

  # Sends activation email.
  def send_activation_email
    UserMailer.account_activation(self).deliver_now
  end

  private
    .
    .
    .
end
```

```ruby
代码清单 10.34: Sending email via the user model object.
# app/controllers/users_controller.rb
 class UsersController < ApplicationController
  .
  .
  .
  def create
    @user = User.new(user_params)
    if @user.save
      @user.send_activation_email
      flash[:info] = "Please check your email to activate your account."
      redirect_to root_url
    else
      render 'new'
    end
  end
  .
  .
  .
end
```
```ruby
代码清单 10.35: Account activation via the user model object.
# app/controllers/account_activations_controller.rb
 class AccountActivationsController < ApplicationController

  def edit
    user = User.find_by(email: params[:email])
    if user && !user.activated? && user.authenticated?(:activation, params[:id])
      user.activate
      log_in user
      flash[:success] = "Account activated!"
      redirect_to user
    else
      flash[:danger] = "Invalid activation link"
      redirect_to root_url
    end
  end
end
```
注意清单10.33消除了user.的使用，因为在Usesr模型里没有这样的变量，它会导致出错：
```ruby
-user.update_attribute(:activated,    true)
-user.update_attribute(:activated_at, Time.zone.now)
+update_attribute(:activated,    true)
+update_attribute(:activated_at, Time.zone.now)
```
(我们可能已经从user替换为self，但是回忆6.2.5节self是在模型里可选的。）它也在User
mailer的调用从@uesr改为self：
```ruby
-UserMailer.account_activation(@user).deliver_now
+UserMailer.account_activation(self).deliver_now
```
这些恰好是那种在简单的重构期间容易被忽视的细节，但是被好的测试集捕捉到了。说到这个，测试集应该是绿色的：
```ruby
代码清单 10.36: 绿色
$ bundle exec rails test
```
账户激活现在完成了，这是一个值得提交的里程碑：
```ruby
$ git add -A
$ git commit -m "Add account activations"
```
## 10.2 密码重置

已经完成了账户激活（所以确认用户的email地址），我们现在到了处理普通用户忘记它们密码的情形的好位置了。正如我们将会看到的，许多步骤是相似的，我们将有几个机会来应用在10.1里学到的东西。开始是困难的，不过；不行账户激活，实现密码重置需要改变我们的视图和两个新的表单（处理email和新密码提交）。

在写代码之前，让我们画出重置密码想要的顺序的模型。我们将通过田间“忘记密码”链接到实例应用程序的登陆表单（图10.7）来开始。“忘记密码”链接将带一个email地址和发送包含密码重置的链接的表单（图10.8）。重置链接链接到重置用户密码（带确认）的表单（图10.9）。

![图10.7：“忘记密码”链接的页面模型](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/login_forgot_password_mockup.png)

![图10.8：“忘记密码”表单的页面模型](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/forgot_password_form_mockup.png)

![图10.9：重置密码表单的页面模型](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/reset_password_form_mockup.png)

在模拟账户激活中，我们的计划是创建密码重置资源，带每个密码重置包含重置口令和相应的重置摘要。主要的顺序像这个：
1. 当用户需要密码重置，通过提交的email地址查找用户。
2. 假如email地址在数据库里存在，生成重置口令和相应的重置摘要。
3. 保存重置摘要带数据库，然后发送包含重置口令和用户email地址的email给用户
4. 当用户点击链接，通过email地址查找用户，然后通过比较重置摘要来验证口令。
5. 假如验证，显示修改密码的表单。

### 10.2.1 密码重置资源
和账户激活一样（10.1.1节），我们第一步是为我们的新资源生成控制器：
```ruby
$ rails generate controller PassswordResets new edit --no-test-framework
```
和在10.1.1节一样，我们包含了跳过生成测试的标志，相反将在10.1.4节生成一个集成测试。

因为我们需要既可以创建新密码重置（图10.8）也可以通过在User模型里改变密码来更新他们（图10.9），我们需要路由到new，create，edit和update。我们能用resources行安排这个，如在清单10。37里显示的一样。
```ruby
代码清单 10.37: Adding a resource for password resets.
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
  resources :account_activations, only: [:edit]
  resources :password_resets,     only: [:new, :create, :edit, :update]
end
```
在清单10.37里的代码准备的REST的路由显示在表10.2里。具体来说，表10.2里的第一个路由通过
```ruby
new_password_reset_path
```
给出了“忘记密码”表单的链接，如在清单10.38和图10.10里见到的一样。

表10.2: 在清单10.37里的密码重置资源提供的REST的路由。
```ruby
代码清单 10.38: Adding a link to password resets.
# app/views/sessions/new.html.erb
 <% provide(:title, "Log in") %>
<h1>Log in</h1>

<div class="row">
  <div class="col-md-6 col-md-offset-3">
    <%= form_for(:session, url: login_path) do |f| %>

      <%= f.label :email %>
      <%= f.email_field :email, class: 'form-control' %>

      <%= f.label :password %>
      <%= link_to "(forgot password)", new_password_reset_path %>
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

![图10.10：带“忘记密码”链接的登陆页面](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/forgot_password_link.png)

为密码重置的数据模型和账户激活（图10.1）的相似。跟随记住口令（8.4节）和账户激活口令（10.1节）的模式，密码重置将用虚拟的为了在重置email里使用的重置口令和相应的为了取回用户的重置摘要组对。假如我们储存未哈希的口令，黑客可以发送重置请求到用户的email地址，然后使用扣和email访问相应的密码重置链接，然后获得账户的控制。为密码重置使用摘要因此是必要的。作为另一个安全其实，我们也计划几小时后让重置链接过节，这需要记录重置发送的时间。成果reset_digest和reset_sent_at属性显示在途10.11里。
![图10.11：带添加了密码重置属性的用户模型](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/forgot_password_link.png)

添加图10.11里的属性的数据迁移显示如下：
```
$ rails generate migration add_reset_to_users reset_digest:string \
> reset_sent_at:datetime
```
```ruby
$ bundle exec rake db:migrate
```

### 10.2.2 密码重置控制器和表单
为了为新的密码重置创建视图，我们将用闲钱的为了创建新的费Active
Record资源的表单模拟，即，为创建新会话的登陆表单（清单8.2），显示在清单10.39为参考。
```ruby
代码清单 10.39: Reviewing the code for the login form.
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
新的密码重置表单和清单10.39有很多相似；最重要的不同是使用不同的资源和URL在调用form_for和没有密码属性。结果显示在清单10.40和图10.12。
```ruby
代码清单 10.40: A new password reset view.
# app/views/password_resets/new.html.erb
 <% provide(:title, "Forgot password") %>
<h1>Forgot password</h1>

<div class="row">
  <div class="col-md-6 col-md-offset-3">
    <%= form_for(:password_reset, url: password_resets_path) do |f| %>
      <%= f.label :email %>
      <%= f.email_field :email, class: 'form-control' %>

      <%= f.submit "Submit", class: "btn btn-primary" %>
    <% end %>
  </div>
</div>
```
![图10.12：“忘记密码”表单](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/forgot_password_form.png)

在途10.12里提交表单，我们需要通过email地址查找用户和用密码重置口令和发送在时间戳更新它的属性。我们然后重定向到根URL，带信息丰富的flash信息。和登陆一样（清单8.9），在无效提交的情形我们重新渲染了new网页，用flash.now信息。结果显现在清单10.41里。
```ruby
代码清单 10.41: A create action for password resets.
# app/controllers/password_resets_controller.rb
 class PasswordResetsController < ApplicationController

  def new
  end

  def create
    @user = User.find_by(email: params[:password_reset][:email].downcase)
    if @user
      @user.create_reset_digest
      @user.send_password_reset_email
      flash[:info] = "Email sent with password reset instructions"
      redirect_to root_url
    else
      flash.now[:danger] = "Email address not found"
      render 'new'
    end
  end

  def edit
  end
end
```
在User模型里的代码和在before_crete回叫函数（清单10.3）里使用的create_activation_digest相似，如清单10.42所见。
```ruby
代码清单 10.42: Adding password reset methods to the User model.
# app/models/user.rb
 class User < ActiveRecord::Base
  attr_accessor :remember_token, :activation_token, :reset_token
  before_save   :downcase_email
  before_create :create_activation_digest
  .
  .
  .
  # Activates an account.
  def activate
    update_attribute(:activated,    true)
    update_attribute(:activated_at, Time.zone.now)
  end

  # Sends activation email.
  def send_activation_email
    UserMailer.account_activation(self).deliver_now
  end

  # Sets the password reset attributes.
  def create_reset_digest
    self.reset_token = User.new_token
    update_attribute(:reset_digest,  User.digest(reset_token))
    update_attribute(:reset_sent_at, Time.zone.now)
  end

  # Sends password reset email.
  def send_password_reset_email
    UserMailer.password_reset(self).deliver_now
  end

  private

    # Converts email to all lower-case.
    def downcase_email
      self.email = email.downcase
    end

    # Creates and assigns the activation token and digest.
    def create_activation_digest
      self.activation_token  = User.new_token
      self.activation_digest = User.digest(activation_token)
    end
end
```
如显示在图10.12里，在这点应用程序的行为为无效email地址已经工作了。为了让应用程序对有效的email地址也工作，我们需要定义密码重置mailer方法。
![图10.13：为无效email地址的“忘记密码”表单](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/invalid_email_password_reset.png)

### 10.2.3 密码重置mailer方法
发送密码重置email的代码显示在清单10.42里如下：
```ruby
UserMailer.password_reset(self).deliver_now
```
密码重置mailer方法需要让这个工作几乎和在10.1.2节开发的账户激活mailer一样。我们首先创建password_reset方法在用户mailer（清单10.43），然后定义视图模板为纯文本email（清单10.44）和HTML email（清单10.45）。
``ruby

sting 10.43: Mailing the password reset link.
# app/mailers/user_mailer.rb
 class UserMailer < ApplicationMailer

    def account_activation(user)
      @user = user
      mail to: user.email, subject: "Account activation"
    end

    def password_reset(user)
      @user = user
      mail to: user.email, subject: "Password reset"
    end
  end
```
``ruby
代码清单 10.44: The password reset plain-text email template.
# app/views/user_mailer/password_reset.text.erb
 To reset your password click the link below:

 <%= edit_password_reset_url(@user.reset_token, email: @user.email) %>

 This link will expire in two hours.

 If you did not request your password to be reset, please ignore this email and
 your password will stay as it is.
```
``ruby
代码清单 10.45: The password reset HTML email template.
# app/views/user_mailer/password_reset.html.erb
 <h1>Password reset</h1>

<p>To reset your password click the link below:</p>

<%= link_to "Reset password", edit_password_reset_url(@user.reset_token,
                                                      email: @user.email) %>

<p>This link will expire in two hours.</p>

<p>
If you did not request your password to be reset, please ignore this email and
your password will stay as it is.
</p>
```
和账户激活email一样（10.1.2节），我们能使用Rails邮件预览来预览密码重置。代码是对清单10.16的完好的模拟，如清单10.46所示。
```ruby
代码清单 10.46: A working preview method for password reset.
# test/mailers/previews/user_mailer_preview.rb
 # Preview all emails at http://localhost:3000/rails/mailers/user_mailer
class UserMailerPreview < ActionMailer::Preview

  # Preview this email at
  # http://localhost:3000/rails/mailers/user_mailer/account_activation
  def account_activation
    user = User.first
    user.activation_token = User.new_token
    UserMailer.account_activation(user)
  end

  # Preview this email at
  # http://localhost:3000/rails/mailers/user_mailer/password_reset
  def password_reset
    user = User.first
    user.reset_token = User.new_token
    UserMailer.password_reset(user)
  end
end
```
有了清单10.46里的代码，HTML和文本email预览显示在途10.14和图10.15里。
![图10.14：HTML版的密码重置邮件预览](http://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/password_reset_html_preview.png)

![图10.15：文本版的密码重置email的预览](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/password_reset_text_preview.png)

在模拟账户激活mailer方法测试（清单10.18），我们将写一个简单的密码重置测试方法，如清单10.47显示。注意我们需要创建密码重置口令为了在视图里使用；不想激活口令，它被before_create回叫函数（清单10.3）的每个用户创建，密码重置口令仅仅当用户成功提交“忘记密码”表单。这将在集成测试里自然的发生（清单10.54），但是在目前的环境，我们需要创建靠手。

```ruby
代码清单 10.47: Adding a test of the password reset mailer method. 绿色
# test/mailers/user_mailer_test.rb
 require 'test_helper'

class UserMailerTest < ActionMailer::TestCase

  test "account_activation" do
    user = users(:michael)
    user.activation_token = User.new_token
    mail = UserMailer.account_activation(user)
    assert_equal "Account activation", mail.subject
    assert_equal [user.email], mail.to
    assert_equal ["noreply@example.com"], mail.from
    assert_match user.name,               mail.body.encoded
    assert_match user.activation_token,   mail.body.encoded
    assert_match CGI::escape(user.email), mail.body.encoded
  end

  test "password_reset" do
    user = users(:michael)
    user.reset_token = User.new_token
    mail = UserMailer.password_reset(user)
    assert_equal "Password reset", mail.subject
    assert_equal [user.email], mail.to
    assert_equal ["noreply@example.com"], mail.from
    assert_match user.reset_token,        mail.body.encoded
    assert_match CGI::escape(user.email), mail.body.encoded
  end
end
```
在这点，测试集应该是绿色的：
```ruby
代码清单 10.48: 绿色
$ bundle exec rails test
```
有了在清单10.43，清单10.44和清单10.45里的代码，提交有效email地址显示在图10.16里的。相应的email显示在服务器日志里，应该看起来像清单10.49。

![图10.16：提交有效email地址的结果](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/valid_email_password_reset.png)

```ruby
代码清单 10.49: A sample password reset email from the server log.
Sent mail to michael@michaelhartl.com (66.8ms)
Date: Thu, 04 Sep 2014 01:04:59 +0000
From: noreply@example.com
To: michael@michaelhartl.com
Message-ID: <5407babbee139_8722b257d04576a@mhartl-rails-tutorial-953753.mail>
Subject: Password reset
Mime-Version: 1.0
Content-Type: multipart/alternative;
 boundary="--==_mimepart_5407babbe3505_8722b257d045617";
 charset=UTF-8
Content-Transfer-Encoding: 7bit


----==_mimepart_5407babbe3505_8722b257d045617
Content-Type: text/plain;
 charset=UTF-8
Content-Transfer-Encoding: 7bit

To reset your password click the link below:

http://rails-tutorial-c9-mhartl.c9.io/password_resets/3BdBrXeQZSWqFIDRN8cxHA/
edit?email=michael%40michaelhartl.com

This link will expire in two hours.

If you did not request your password to be reset, please ignore this email and
your password will stay as it is.
----==_mimepart_5407babbe3505_8722b257d045617
Content-Type: text/html;
 charset=UTF-8
Content-Transfer-Encoding: 7bit

<h1>Password reset</h1>

<p>To reset your password click the link below:</p>

<a href="http://rails-tutorial-c9-mhartl.c9.io/
# password_resets/3BdBrXeQZSWqFIDRN8cxHA/
edit?email=michael%40michaelhartl.com">Reset password</a>

<p>This link will expire in two hours.</p>

<p>
If you did not request your password to be reset, please ignore this email and
your password will stay as it is.
</p>
----==_mimepart_5407babbe3505_8722b257d045617--
```
### 10.2.4 重置密码
为了get表单的链接
```ruby
http://example.com/password_resets/3BdBrXeQZSWqFIDRN8cxHA/edit?email=foo%40bar.com
```
工作，我们需要一个重置密码的表单。任务和通过用户编辑视图（清单9.2）相似，但是在这个例子，仅有密码和密码确认框。有另外的复杂，不过：我们想要通过email地址找到用户，这意味着我们需要它的值在edit和update动作里。email将自动在edit动作里可用，因为它出现在上面的链接里，但是我们提交表单后他的值将会失去。解决方法是使用隐藏区域来方式（但是不显示）email在页面上，然后提交它和剩余表单信息一样。结果显示在清单10.50.
```ruby
代码清单 10.50: The form to reset a password.
# app/views/password_resets/edit.html.erb
 <% provide(:title, 'Reset password') %>
<h1>Reset password</h1>

<div class="row">
  <div class="col-md-6 col-md-offset-3">
    <%= form_for(@user, url: password_reset_path(params[:id])) do |f| %>
      <%= render 'sha红色/error_messages' %>

      <%= hidden_field_tag :email, @user.email %>

      <%= f.label :password %>
      <%= f.password_field :password, class: 'form-control' %>

      <%= f.label :password_confirmation, "Confirmation" %>
      <%= f.password_field :password_confirmation, class: 'form-control' %>

      <%= f.submit "Update password", class: "btn btn-primary" %>
    <% end %>
  </div>
</div>
```
注意清单10.50使用表单辅助方法
```ruby
hidden_field_tag :email, @user.email
```
不是
```ruby
f.hidden_field :email, @user.email
```
因为重置链接把email放在params[:email]，然而后者将把它放在params[:user][:email]。

为了让表单渲染，我们需要定义@user变量在密码重置控制器的edit动作。也账户激活一样（清单10.29），这需要查找用户相应的email地址在params[:email]。我们然后需要确认用户是有效的，例如，它存在，是激活的，被根据重置口令params[:id]验证。因为有效的@user的存在被需要在edit和update动作里，我们将放代码到查找和验证它在几个前置过滤，如清单10.51所示。
```ruby
代码清单 10.51: The edit action for password reset.
# app/controllers/password_resets_controller.rb
 class PasswordResetsController < ApplicationController
  before_action :get_user,   only: [:edit, :update]
  before_action :valid_user, only: [:edit, :update]
  .
  .
  .
  def edit
  end

  private

    def get_user
      @user = User.find_by(email: params[:email])
    end

    # Confirms a valid user.
    def valid_user
      unless (@user && @user.activated? &&
              @user.authenticated?(:reset, params[:id]))
        redirect_to root_url
      end
    end
end
```
在清单10.51里，比较用法
```ruby
authenticated?(:reset, params[:id])
```
到
```ruby
authenticated?(:activation, params[:id])
```
在清单10.29里。一起，这三个使用完整的验证方法显示在表10.1.

有了一万行的代码，从清单10.49里的链接应该渲染密码重置表单。结果显示在图10.17.
![图10.17：密码重置表单](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/password_reset_form.png)

为了定义update动作相应的edit动作在清单10.51里，我们需要考虑四种情形：过期的密码重置，成功的更新，失败的更新（由于无效密码），和失败的更新（刚开始看上去“成功”）由于空密码和确认。第一种情形应用到edit和update动作，所以逻辑上属于前置过滤（清单10.52）。下面的两种情形相应的两个分支在主要的if语句显示在清单10.52里。因为编辑表单是修改Active Record模型对象（例如，用户），我们能依赖分享的视图片段从清单10.50来渲染错误信息。仅仅的例外是密码是空的，现在被我们的User模型允许（清单9.10）和所以徐璈被抓住和显式地处理。我们的方法在这个情况是添加错误直接到@user对象的错误信息：
```ruby
@user.errors.add(:password, "can't be empty")
```
```ruby
代码清单 10.52: The update action for password reset.
# app/controllers/password_resets_controller.rb
 class PasswordResetsController < ApplicationController
  before_action :get_user,         only: [:edit, :update]
  before_action :valid_user,       only: [:edit, :update]
  before_action :check_expiration, only: [:edit, :update]

  def new
  end

  def create
    @user = User.find_by(email: params[:password_reset][:email].downcase)
    if @user
      @user.create_reset_digest
      @user.send_password_reset_email
      flash[:info] = "Email sent with password reset instructions"
      redirect_to root_url
    else
      flash.now[:danger] = "Email address not found"
      render 'new'
    end
  end

  def edit
  end

  def update
    if params[:user][:password].empty?
      flash.now[:danger] = "Password can't be empty"
      render 'edit'
    elsif @user.update_attributes(user_params)
      log_in @user
      flash[:success] = "Password has been reset."
      redirect_to @user
    else
      render 'edit'
    end
  end

  private

    def user_params
      params.require(:user).permit(:password, :password_confirmation)
    end

    # Before filters

    def get_user
      @user = User.find_by(email: params[:email])
    end

    # Confirms a valid user.
    def valid_user
      unless (@user && @user.activated? &&
              @user.authenticated?(:reset, params[:id]))
        redirect_to root_url
      end
    end

    # Checks expiration of reset token.
    def check_expiration
      if @user.password_reset_expi红色?
        flash[:danger] = "Password reset has expi红色."
        redirect_to new_password_reset_url
      end
    end
end

```
在清单10.52里的实现委托了密码重置过期的测试到User模型通过代码
```ruby
@user.password_reset_expi红色?
```
为了让这个工作，我们需要定义password_reset_expi红色?方法。如从10.2.3节在email模板里现实的。我们将考虑密码重置是过期的假如它在两小时前发送。我们能用Ruby如下表达：
```ruby
reset_sent_at < 2.hours.ago
```
这里可能会引起困惑，假如你读<为“小于”，因为然后他听起来想“密码重置发送小于两小时以前”，和我们想要的正好相反。在这个环境，最好读为“早于”，它给一些东西好像“密码重置发送早于两个小时之前”。那是我们想要的，它导致password_reset_expi红色?放过在清单10.53.(为了正式的表面比较是正确的，看10.6节的证据）
```ruby
代码清单 10.53: Adding password reset methods to the User model.
# app/models/user.rb
 class User < ActiveRecord::Base
  .
  .
  .
  # Returns true if a password reset has expi红色.
  def password_reset_expi红色?
    reset_sent_at < 2.hours.ago
  end

  private
    .
    .
    .
end
```
有了清单10.53的代码，update动作在清单10.52里应该工作了。结果为无效和有效的提交显示在途10.18和图10.19，各自。（缺少耐心等两小时，我们将在第三个分支里覆盖，这留下作为练习（10.5节）。）

![图10.18：失败的密码重置](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/password_reset_failure.png)

![图10.19：成功密码重置](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/password_reset_successful.png)

### 10.2.5 密码重置测试

在这节，我们将写一个集成测试覆盖三个分支的两个在清单10.52，无效和有效的提交。（无上面提到的，测试第三个分支留下来作为练习。）我们将开始通过生成测试文件为密码重置：
```ruby
$ rails generate integration_test password_resets
      invoke  test_unit
      create    test/integration/password_resets_test.rb
```
测试密码重置的步骤广泛地和测试账户激活从清单10.31里的代码相似，尽管有一个不同在外部：我们第一次方法“忘记密码”表单和提交无效和然后有效emaild地址，后者的创建密码重置口令和发送了重置email。我们然后访问链接从email，再次提交无效和有效的信息，确认正确的行为在每种情形。结果测试，显示在清单10.54里，是非常好的读代码的练习。
```ruby
代码清单 10.54: An integration test for password resets.
# test/integration/password_resets_test.rb
 require 'test_helper'

class PasswordResetsTest < ActionDispatch::IntegrationTest
  def setup
    ActionMailer::Base.deliveries.clear
    @user = users(:michael)
  end

  test "password resets" do
    get new_password_reset_path
    assert_template 'password_resets/new'
    # Invalid email
    post password_resets_path, password_reset: { email: "" }
    assert_not flash.empty?
    assert_template 'password_resets/new'
    # Valid email
    post password_resets_path, password_reset: { email: @user.email }
    assert_not_equal @user.reset_digest, @user.reload.reset_digest
    assert_equal 1, ActionMailer::Base.deliveries.size
    assert_not flash.empty?
    assert_redirected_to root_url
    # Password reset form
    user = assigns(:user)
    # Wrong email
    get edit_password_reset_path(user.reset_token, email: "")
    assert_redirected_to root_url
    # Inactive user
    user.toggle!(:activated)
    get edit_password_reset_path(user.reset_token, email: user.email)
    assert_redirected_to root_url
    user.toggle!(:activated)
    # Right email, wrong token
    get edit_password_reset_path('wrong token', email: user.email)
    assert_redirected_to root_url
    # Right email, right token
    get edit_password_reset_path(user.reset_token, email: user.email)
    assert_template 'password_resets/edit'
    assert_select "input[name=email][type=hidden][value=?]", user.email
    # Invalid password & confirmation
    patch password_reset_path(user.reset_token),
          email: user.email,
          user: { password:              "foobaz",
                  password_confirmation: "barquux" }
    assert_select 'div#error_explanation'
    # Empty password
    patch password_reset_path(user.reset_token),
          email: user.email,
          user: { password:              "",
                  password_confirmation: "" }
    assert_not flash.empty?
    assert_template 'password_resets/edit'
    # Valid password & confirmation
    patch password_reset_path(user.reset_token),
          email: user.email,
          user: { password:              "foobaz",
                  password_confirmation: "foobaz" }
    assert is_logged_in?
    assert_not flash.empty?
    assert_redirected_to user
  end
end

```

在清单10.54里大多数的相反已经在先前显示过；仅仅新的东西是测试的input标签：
```ruby
assert_select "input[name=email][type=hidden][value=?]", user.email
```
这确认有input标签有正确的名字，（隐藏）类型，和email地址：
```ruby
<input id="email" name="email" type="hidden" value="michael@example.com" />
```
有了在清单10.54里的代码，我们的测试集应该是绿色的：
```ruby
代码清单 10.55: 绿色
$ bundle exec rails test
```

## 10.3 在生产环境的Email

作为我们工作的压顶石头关于账户激活和密码提醒，在这节我们将配置我们的应用程序以便它能真的在生产环境发送email。我们将用免费得发送邮件服务喀什，然后配置和部署我们的应用程序。
为了在生产环境发送email，我们将使用SendGrid，它作为Heroku确认账户的讨价是可用的。（这要求额外的信用卡信息到你的Heroku账户，但是不收费当确认账户时）为了我们的目的，“starter”tier（限制每天发送200份email，但是不收费）是最适合的。我们能把它按照如下添加到我们的app：
```ruby
$ heroku addons:create sendgrid:starter
```
(这可能失败在系统用旧版的Heroku's的命令行接口。在这种情形，升级到最近的Heroku或者实施旧的heroku语法heroku
addons:add sendgrid:starter)

为了配置我们的应用程序使用SendGrid，我们需要为我们的生产环境填出SMTP设置。如显示在青岛10.56里的，以也必须定义host变量带你的产品的网站。
```ruby
代码清单 10.56: Configuring Rails to use SendGrid in production.
# config/environments/production.rb
 Rails.application.configure do
  .
  .
  .
  config.action_mailer.raise_delivery_errors = true
  config.action_mailer.delivery_method = :smtp
  host = '<your heroku app>.herokuapp.com'
  config.action_mailer.default_url_options = { host: host }
  ActionMailer::Base.smtp_settings = {
    :address        => 'smtp.sendgrid.net',
    :port           => '587',
    :authentication => :plain,
    :user_name      => ENV['SENDGRID_USERNAME'],
    :password       => ENV['SENDGRID_PASSWORD'],
    :domain         => 'heroku.com',
    :enable_starttls_auto => true
  }
  .
  .
  .
end
```
在清单10.56里的email配置包含user_name和SendGrid账户的密码，但是注意他们是通过ENV环境变量进入的，而不是硬编码。这是在生产应用程序的最佳实践，它是为了安全的愿意应从不暴露敏感信息，例如原始密码在源代码里。目前的情形，这些变量被自动配置通过SendGrid插件，但是我们将在11.4.4节里看见我们不得不自己定义它们。万一你好奇，你可以浏览环境变量使用在清单10.56里显示的入学：
```ruby
$ heroku config:get SENDGRID_USERNAME
$ heroku config:get SENDGRID_PASSWORD
```

在这点，你应该合并主题分支进入主分支：
```ruby
$ bundle exec rails test
$ git add -A
$ git commit -m "Add password resets & email configuration"
$ git checkout master
$ git merge account-activation-password-reset
```
然后推送到远程仓库，然后部署到Heroku：
```ruby
$ bundle exec rails test
$ git push
$ git push heroku
$ heroku run rake db:migrate
```
一旦Heroku部署结束，试着注册示例程序在生产环境，使用你自己的email地址。你应该得到一份激活email如在10.1.1节（图10.20）实现的一样。假如你然后忘记了或假装忘记了你的密码，你能再10.2节开发的重置（图10.21）。
![图10.20：在生产发送的账户激活邮件](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/activation_email_production.png)
![图10.21:在生产里的密码重置邮件](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/reset_email_production.png)

## 10.4 总结

随着添加了账户激活和密码重置，我们的样例应用程序注册，登陆，退出机制是完整的，专家级的。剩余的教程将在这个基础上建立一个类似Twitter的微博网站（第11章）和关注用户（第12章）。在这个过程，我们将学习一些Rails最有力的特性，包含图片上传，自定义数据库查询，和高级的数据模型化用has_many和has_many :through.

### 10.4.1 我们在这章学到的
* 类似会话，账户激活能被模型化作为资源尽管没有Active Record对象
* Rails能生产Active Mailer和视图来发送邮件。
* Action Mailer支持纯文本和HTML邮件
* 和普通的动作和视图一样，实例变量定义在mailer动作里的是在mailer视图里可用
* 像会话和账户激活，密码重置能被作为资源模型化，尽管不是Active Record对象
* 账户激活和密码重置使用生成的口令创建唯一的URL为了激活用户或重置密码，各自地。
* mailer测试和集成测试对于确认用户Mailer行为是有效的
* 我们能在生产环境使用SendGrid发送邮件

## 10.5 联系

1.为国企的密码重置分支在青岛10.52里的写集成测试，通过天在清单10.57的模板。（这段代码引入response.body,它返回完整的Html页面）有许多方法测试解释的结果，但是青岛10.57建议的方法是（大小写敏感）检查收到的页面包含单词expi红色。
2.
现在所有用户显示在用户注意在/users通过/users/:id是可见的，但是仅仅显示激活用户的合理的。安排这个行为通过填充青岛10.58里的模板。（这使用了Active
Recordwhere方法，我们将在11.3.3节学习更多）附加题：为/users和/users/:id写集成测试。
3.
在清单10.42里，activate和create_reset_digest方法创建两个调用update_attribute，每个要求独自数据库交易。通过填充显示在清单10.59里的代码，代替没对update_attribute调用带一个调用update_columns,它仅仅查询一次数据库。这些变化后，确认测试集是绿色的。
```ruby
代码清单 10.57: A test for an expi红色 password reset. 绿色
# test/integration/password_resets_test.rb
 require 'test_helper'

class PasswordResetsTest < ActionDispatch::IntegrationTest

  def setup
    ActionMailer::Base.deliveries.clear
    @user = users(:michael)
  end
  .
  .
  .
  test "expi红色 token" do
    get new_password_reset_path
    post password_resets_path, password_reset: { email: @user.email }

    @user = assigns(:user)
    @user.update_attribute(:reset_sent_at, 3.hours.ago)
    patch password_reset_path(@user.reset_token),
          email: @user.email,
          user: { password:              "foobar",
                  password_confirmation: "foobar" }
    assert_response :redirect
    follow_redirect!
    assert_match /FILL_IN/i, response.body
  end
end
```
```ruby
代码清单 10.58: A template for code to show only active users.
# app/controllers/users_controller.rb
 class UsersController < ApplicationController
  .
  .
  .
  def index
    @users = User.where(activated: FILL_IN).paginate(page: params[:page])
  end

  def show
    @user = User.find(params[:id])
    redirect_to root_url and return unless FILL_IN
  end
  .
  .
  .
end

```

```ruby
代码清单 10.59: A template for using update_columns.
# app/models/user.rb
 class User < ActiveRecord::Base
  attr_accessor :remember_token, :activation_token, :reset_token
  before_save   :downcase_email
  before_create :create_activation_digest
  .
  .
  .
  # Activates an account.
  def activate
    update_columns(activated: FILL_IN, activated_at: FILL_IN)
  end

  # Sends activation email.
  def send_activation_email
    UserMailer.account_activation(self).deliver_now
  end

  # Sets the password reset attributes.
  def create_reset_digest
    self.reset_token = User.new_token
    update_columns(reset_digest:  FILL_IN,
                   reset_sent_at: FILL_IN)
  end

  # Sends password reset email.
  def send_password_reset_email
    UserMailer.password_reset(self).deliver_now
  end

  private

    # Converts email to all lower-case.
    def downcase_email
      self.email = email.downcase
    end

    # Creates and assigns the activation token and digest.
    def create_activation_digest
      self.activation_token  = User.new_token
      self.activation_digest = User.digest(activation_token)
    end
end
```
## 10.6 过期比较的证据

