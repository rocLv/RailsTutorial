# 第六章 用户模型

在第五章，我们做了个临时的注册页面（5.4节）。在接下来的五章，我们将完成用户注册页面里暗示的承诺。本章我们将通过创建用户模型开始艰难的第一步，包括数据存储。在第七章，我们也会为在我们网站注册的用户创建一个用户信息页面。一旦用户可以注册，我们也准备实现用户登陆和退出的功能（第八章），在第九章（9.2.1节）我们将学习怎样防止未未授权用户登陆。最后，在第十章，我们将添加账户激活功能（通过确认有效的email地址）和密码重置功能。综合起来，从第六章到第十章我们开发了一个完整的Rails登陆和授权系统。正如你可能知道的，在Rails社区有许多用户验证解决方案，注6.1解释了为什么，起码在刚学Rails的时候，建立自己的一套用户登陆验证系统可能是更好的想法。

    注6.1 建立自己的授权系统

    实际上，几乎所有的网站都需要用户登陆和授权系统。所以大多数WEB框架都有很多这样的系统可供选择，Rails也不例外。认证和授权系统包括Clearance、Authlogic、Devise和CanCanCan（和建立在OpenID和OAuth之上的非Rails专用方案）。你肯定想问为什么我们应该重新发明轮子？为什么不使用现成的方案而是创建自己的用户认证和授权系统？

    这是因为实际的经验显示授权系统在大多数网站都有自定义扩展的需求。修改第三方产品常常需要比从零开始构建系统花费更多的工作。另外，现成的系统可能是“黑盒”，有潜在的迷一样的内部结构。当你在编写自己的系统时，你更可能充分理解它。而且，近来的Rails（6.3节）让写一个自定义授权系统更加容易。最后，假如你最终还是使用了第三方系统，但是一旦你曾经自己创建过一个类似的系统，你会更好的理解第三方系统，并在必要的时候可以修改它。

## 6.1 用户模型
尽管接下来三章的终极目标是为我们的网站创建注册页面（原型如图6.1），但是现在接受新用户信息几乎还没什么用：现在还没有存放用户数据的地方。因此，用户注册的第一步是创建接收和储存用户信息的数据结构。

![图6.1：用户注册页面原型](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/signup_mockup_bootstrap.png)

在Rails里，数据模型默认的数据结构叫模型（Model，MVC中的M），足够自然。默认的Rails解决持久性问题是用数据库来长期保存数据，和数据库交互默认的库是Activ Record。ApplicationRecord有许多用来创建、保存和查找数据对象的方法。所有的查找都不需要使用关系数据库使用的结构化查询语句（SQL），而且Rails有一个特性叫migration，允许用纯Ruby写数据定义，所以也不必学习SQL数据定义语言（DDL）。结果是Rails把和数据存储细节几乎隔离开来。
本书在开发环境使用SQLite，生产环境用PostgreSQL（通过Heroku）。这个主题我们已经跨的很远了，跨到我们几乎不得不考虑Rails怎样储存数据，甚至需要考虑生产环境的应用程序怎样储存数据。

和往常一样，假如你一直使用Git做版本控制，现在是时候为模型化用户创建一个主题分支：

```terminal
$ git checkout master
$ git checkout -b modeling-users
```

### 6.1.1 数据库迁移
你可能回忆起在4.4.5节我们已经自定义过一个User类，它有name和email两个属性。那个例子作为一个有用的例子缺少严格的持续性的属性：当我们在控制台创建User对象后，当我们退出时这个对象就消失了。我们这节的目标是为用户创建模型，让它不会如此容易就消失。

和4.4.5节里的User类一样，我们将通过两个属性name和email来模型化用户，email将作为用户唯一的用户名。（我们在6.3节加入密码属性）在代码清单4.13里，我们用attr_accessor方法实现了：
```ruby
class User
  attr_accessor :name, :email
  .
  .
  .
end

```
相反，当使用Rails来模型化用户时，我们不需要严格地识别属性的读写权限。如同上面简单提到过的，默认使用关系数据库储存数据，它包含由数据行组成的表，每行由属性组成列构成。例如，为了储存用户的name和emai，我们使用name和email列创建一个用户表（每行对应一个用户）。这种的表的例子在图6.2里显示，对应的数据模型显示在图6.3里。（图6.3只是草图，完整的数据模型如图6.4）。通过命名列name和emai，我们让ApplicationRecord为我们查找User对象的属性。

![图6.2：用户表里的示例数据](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/users_table.png)

![图6.3：用户数据模型的草图](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/users_table.png)

你可能想起代码清单5.28，我们创建了一个User控制器（带动作new），使用命令
```
 $ rails generate controller Users new
```
自动生成控制器代码。生成模型的命令类似，是`rails generate model`，我们用来生成用户模型，具有name和email属性，如代码清单6.1所示。

```terminal
代码清单 6.1： 生成用户模型。
$ rails generate model User name:string email:string
      invoke  active_record
      create    db/migrate/20151224010738_create_users.rb
      create    # app/models/user.rb
      invoke    test_unit
      create      test/models/user_test.rb
      create      test/fixtures/users.yml
```
（注意，和使用复数的控制器惯例不同，模型名字是单数的：用户们的控制器，但是用户的模型。）通过传递可选参数name:string和email:string，我们告诉Rails我们想要添加那些属性、这些属性应该是那种数据类型（这里是String）。比较这个和代码清单3.4和5.28里的动作名称）

代码清单6.1里generate命令的创建了一个叫做迁移(migration)的新文件。迁移提供了一种增量改变数据结构的方法，我们所有的数据模型都可以使用它来根据需求改变。在用户模型这个例子里，迁移是通过模型生成脚本自动创建的；它创建了用户表，有两列：name和email，如代码清单6.2所示。（我们将在6.2.5节开始学习怎样从零开始创建数据迁移）
```ruby
代码清单 6.2： 为User模型创建的数据迁移（为了创建users表）。
# db/migrate/[timestamp]_create_users.rb
class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :name
      t.string :email

      t.timestamps null: false
    end
  end
end
```

注意迁移文件名称有一个时间戳前缀，依据是迁移生成的时间。在早期的迁移，文件名是递增的数据，但是假如多个程序员合作开发时容易引起冲突。摒弃不太可能发生的在同一秒生成迁移的场景，使用时间戳避免了这种冲突。

迁移自身包含了change方法，决定了对数据库所作的改变。在代码清单6.2里，change使用Rails create_table的方法，在数据库里创建了一个表储存用户数据。create_table方法接受块（4.3.2节）作为参数，带一个块变量，在这里是t。在块内部，create_table方法使用t对象在数据库里创建了name和email列，两个都是string类型。这里表名是复数（users），即使模型名称是单数（User）。它反映了Rails遵循的语法惯例：模型代表单个用户，然而数据表包含了许多用户。在块里，最后的一行， t.timestamps null: false，是特殊的命令，创建两个魔法列叫做created_at和updated_at，是自动记录指定用户创建和更新的时间。（我们将在6.1.3节里看见魔法列的实际例子）。在代码清单6.2里迁移代表的完整数据模型如图5.6.4所示。（注意额外的魔法列，我们在图6.3里没有显示）

![图6.4：代码清单6.2产生的用户数据模型](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_model_initial_3rd_edition.png)

我们可以运行迁移，知名的“迁移起来”，使用rails命令（注3.1）如下：

```
$ bundle exec rails db:migrate
```
（你可能回忆起来我们在2.2节类似的环境运行过这个命令）db:migrate首次运行，它创建了一个名为db/development.sqlite3的文件，它是SQLite数据库。我们可以用[SQLite数据库浏览器](http://sqlitebrowser.org/)打开development.sqlite3文件。
（假如你正使用云IDE，你应该先把数据库文件下载到本地，如图6.5所示）。结果如图6.6所示）；和图6.4比较，你可能注意到在图6.6有一列，在迁移文件里却没有：id列。如同在2.2节简单提过的这个列是自动创建的，Rails用它来识别每一行数据。

![从云IDE下载文件](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/sqlite_download.png)

![图6.6：SQLite数据库浏览器显示新的用户表](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/sqlite_download.png)

许多迁移（包括本书中所有的）是可逆的，这意味着我们可以“迁移回去”，用一个Rails任务还原它们，叫做db:rollback:

```
$ bundle exec rails db:rollback
```
(参考注3.1查看另一个有用的技术，逆迁移。）在后台，这个命令执行drop_table从数据库移除用户表。这样工作的原理是change方法指导drop_table是create_table的逆操作，这意味着回滚操作可以容易地被推理出来。在这个不可逆转的迁移例子里，例如移除数据库的一列，这样就有必要分别定义up和down方法替代唯一的change方法。阅读[Rails Guides](http://guides.rubyonrails.org/migrations.html)获得更多信息。

假如你回滚了数据库，在继续学习前再次迁移起来：

```
$ bundle exec rails db:migrate
```

### 6.1.2 模型文件

我们已经在代码清单6.1生成了User模型、生成了一个迁移文件（代码清单6.2），我们在图6.6里看见了运行这个迁移的结果：它更新了development.sqlite3的文件，通过创建用户表，表里有列id，name，email，created_at和updated_at。代码清单6.1也创建了模型本身。这节剩余部分用来理解它。

我们看看User模型的代码，它# app/models目录里的user.rb中。它是，委婉地说，很紧凑的（代码清单6.3）。

···ruby
代码清单 6.3： 全新的User 模型。
# # app/models/user.rb

class User < ApplicationRecord
end

```
回忆在4.4.2节，语法class User < ApplicationRecord的意思是User类继承于ApplicationRecord，以便User模型自动有了ApplicationRecord类的所有功能。当然，这个学问出发我们知道ApplicationRecord包含什么才有用，所有让我们开始一些实际的例子。

### 6.1.3 创建User对象

如在第四章里，我们用Rails控制台探索数据模型。因为我们不想在交互的时候修改数据库，所以我们在沙盒里运行控制台：

```terminal
$ rails console --sandbox   #简写为：rails c -s
Loading development environment in sandbox
Any modifications you make will be rolled back on exit
>>
```

正如帮助信息表明的“你所做的任何改变退出时将会回滚”。当在沙盒里与数据库交互后，在退出控制台的时候控制台会“回滚”（还原）期间发生的任何数据库变化。

在4.4.5节的控制台session里，我们用User.new创建了一个用户对象，那时我们必须包含代码清单4.13里的sampel_user.rb文件后才可以。对于模型来说，情形有点不同。如果你回忆在4.4.4节中，Rails控制台自动加载了Rails环境，其中就包含了模型。这意味着我们可以无需其他操作就可以创建新用户对象：
```terminal
>> User.new
=> #<User id: nil, name: nil, email: nil, created_at: nil, updated_at: nil>
```

我们看见了控制台输出的User对象。

当我们不带参数调用new函数时，User.new返回一个所有属性都是nil的对象。在4.4.5节，我们设计了示例User类，用初始化的哈希来设置对象属性；那个设计是受ApplicationRecord启发，它允许对象以同样的方法初始化：

```
>> user = User.new(name: "Michael Hartl", email: "mhartl@example.com")
=> #<User id: nil, name: "Michael Hartl", email: "mhartl@example.com",
created_at: nil, updated_at: nil>
```

这里我们看见name和email属性已经和设想的那样设置好了。

有效性的概念对于理解ApplicationRecord模型对象是重要的。我们将在6.2节里更加深入地探索这个主题。但是目前对刚刚初始化的User对象来说有效性还没什么意义，我们可以通过调用逻辑函数valid?方法来验证一下：

```
>> user.valid?
true
```

到目前为止，我们还没有真正接触数据库：User.new仅在内存里创建了一个对象，然而user.valid?只检查对象是否有效。为了把User对象保存到数据库，我们需要在user变量上调用save方法：

```
>> user.save
(0.2ms)  begin transaction
  User Exists (0.2ms)  SELECT  1 AS one FROM "users"  WHERE LOWER("users".
  "email") = LOWER('mhartl@example.com') LIMIT 1
  SQL (0.5ms)  INSERT INTO "users" ("created_at", "email", "name", "updated_at)
   VALUES (?, ?, ?, ?)  [["created_at", "2014-09-11 14：32：14.199519"],
   ["email", "mhartl@example.com"], ["name", "Michael Hartl"], ["updated_at",
  "2014-09-11 14：32：14.199519"]]
   (0.9ms)  commit transaction
=> true
```

save方法假如成功返回true，否则返回false。（现在，保存应该成功，因为仍然没有有效性验证；我们将在6.2节里看见有时会失败。）为了提供参考，Rails控制台也显示和user.save相对应的SQL命令（即，INSERT INTO "users" ...)。我们在本书里几乎不需要使用原始的SQL，我将从现在开始忽略讨论SQL命令，但是你可以通过阅读ApplicationRecord相应的SQL学到很多相关的知识。让我们看看我们的save做了些什么：

```
>> user
=> #<User id: 1, name: "Michael Hartl", email: "mhartl@example.com",
created_at: "2014-07-24 00：57：46", updated_at: "2014-07-24 00：57：46">

```
我们看见已经为user的ID赋值为1，魔法列也已经被赋值为当前的时间和日期。现在，创建和更新时间戳是一样的，我们在6.1.5节会看见它们有时也会不同。

正如4.4.5节里User类一样，User模型的实例允许通过“.”访问它们的属性：

```
>> user.name
=> "Michael Hartl"
>> user.email
=> "mhartl@example.com"
>> user.updated_at
=> Thu, 24 Jul 2014 00：57：46 UTC +00：00
```

正如我们将在第七章看见的一样，前面的例子一样分创建和保存两步来保存数据模型，但是ApplicationRecord也让我们一步完成，使用User.create:

```
>> User.create(name: "A Nother", email: "another@example.org")
#<User id: 2, name: "A Nother", email: "another@example.org", created_at:
"2014-07-24 01：05：24", updated_at: "2014-07-24 01：05：24">
>> foo = User.create(name: "Foo", email: "foo@bar.com")
#<User id: 3, name: "Foo", email: "foo@bar.com", created_at: "2014-07-24
01：05：42", updated_at: "2014-07-24 01：05：42">
```

注意User.create，不是返回true和false，而是返回User对象本身，我们也可以选择分配一个变量（例如上面的第二个例子赋值给foo）。
与create相对应的是destroy：

```
>> foo.destroy
=> #<User id: 3, name: "Foo", email: "foo@bar.com", created_at: "2014-07-24
01：05：42", updated_at: "2014-07-24 01：05：42">
```
像create，destroy返回对象，尽管我回忆不起来曾经用过destroy的返回值。另外，删除的对象仍会在内存里存在：

```
>> foo
=> #<User id: 3, name: "Foo", email: "foo@bar.com", created_at: "2014-07-24
01：05：42", updated_at: "2014-07-24 01：05：42">
```
那么我们怎么知道是否已经删除了？而且对于已经保存的和没有删除的对象，我们怎样才能从数据库里撤回用户？为了回答这些问题，我们需要学习怎样使用ApplicationRecord查找用户对象。

### 6.1.4 查找用户
ApplicationRecord提供了几个查找用户的方法。让我们查找第一个用户，然后确认第三个用户（foo）已经删除了。我们从存在的用户开始：

```
>> User.find(1)
=> #<User id: 1, name: "Michael Hartl", email: "mhartl@example.com",
created_at: "2014-07-24 00：57：46", updated_at: "2014-07-24 00：57：46">
```
这里我们给User.find方法传递了第一个用户的id；ApplicationRecord返回id为1的用户。

让我们看看id为3的用户是否仍然在数据库里：
```
>> User.find(3)
ActiveRecord::RecordNotFound: Couldn't find User with ID=3
```

因为我们在6.1.3节删除了我们的第三个用户，Active Record在数据库里找不到他。因此find抛出异常，这是程序里表示例外事件的一种方式--在这里一个不存在的Active Record id引起find抛出ActiveRecord::RecordNotFound异常。

除了普通的find，Active Record也允许我们通过特殊的属性查找用户：

```
>> User.find_by(email: "mhartl@example.com")
=> #<User id: 1, name: "Michael Hartl", email: "mhartl@example.com",
created_at: "2014-07-24 00：57：46", updated_at: "2014-07-24 00：57：46">
```

因为我们后面会使用email地址作为用户登陆时使用的用户名，因此当我们学习怎样让用户登陆我们网站时，这类型的find将会有用（第七章）。假如你担心用户量很多时find_by效率比较低，你已经领先了。我们后面会讨论这些问题，在6.2.5节中我们通过为数据库构建索引来解决。

我们将通过几个更普通的查找用户的方法来结束本小结。首先，是first：
```ruby
>> User.first
=> #<User id: 1, name: "Michael Hartl", email: "mhartl@example.com",
created_at: "2014-07-24 00：57：46", updated_at: "2014-07-24 00：57：46">
```
自然，first返回数据库里第一个用户；还有all：
```ruby
>> User.all
=> #<ActiveRecord::Relation [#<User id: 1, name: "Michael Hartl",
email: "mhartl@example.com", created_at: "2014-07-24 00：57：46",
updated_at: "2014-07-24 00：57：46">, #<User id: 2, name: "A Nother",
email: "another@example.org", created_at: "2014-07-24 01：05：24",
updated_at: "2014-07-24 01：05：24">]>
```
正像你从控制台看见的，User.all以数组形式返回数据库里所有的用户数据。它们都是类ActiveRecord::Relation的实例。

### 6.1.5 更新用户

一旦我们创建了用户，我们可能经常需要更新他们的信息。有两个基础的方法实现这个。首先，我们可以单独为某个属性赋值，正如我们在4.4.5节所做的：

```ruby
>> user           # 只是为了回忆一下user的属性
=> #<User id: 1, name: "Michael Hartl", email: "mhartl@example.com",
created_at: "2014-07-24 00：57：46", updated_at: "2014-07-24 00：57：46">
>> user.email = "mhartl@example.net"
=> "mhartl@example.net"
>> user.save
=> true
```

注意最后把变化写入数据库是必须的。我们能通过使用reload看看有没有保存，reload会依据数据库信息重新加载对象：

```
>> user.email
=> "mhartl@example.net"
>> user.email = "foo@bar.com"
=> "foo@bar.com"
>> user.reload.email
=> "mhartl@example.net"
```

既然我们已经通过运行user.save更新了用户，魔法列的值也自动改变了，如我们在6.1.3节说过的：

```ruby
>> user.created_at
=> "2014-07-24 00：57：46"
>> user.updated_at
=> "2014-07-24 01：37：32"
```
另外一个可以同时更新几个属性的主要方法是使用update_attributes:

```ruby
>> user.update_attributes(name: "The Dude", email: "dude@abides.org")
=> true
>> user.name
=> "The Dude"
>> user.email
=> "dude@abides.org"
```

update_attributes方法以哈希作为参数，假如成功的话会依次执行update和save（保存成功则返回true）。注意假如无论哪个属性验证失败，例如当需要密码才能保存记录（如6.3）, update_attributes都会失败，返回false。假如我们只需要更新其中某个属性时，可以使用update_attribute来绕过这个限制：

```ruby
>> user.update_attribute(:name, "The Dude")
=> true
>> user.name
=> "The Dude"
```

## 6.2 用户验证
我们在6.1节创建的用户模型拥有name和email两个属性。但是它们是非常普通的：现在任何字符串（包含一个空字符串）对它们来说都是有效的。但是无论是name还是email都不应该是任意字符串。例如，name应该非空，email应该匹配一定的格式。而且因为我们准备使用email作为用户登陆时唯一的用户名，
我们不允许数据库里有重复的email。

简而言之，我们要求name和email两个属性要满足一定的限制。ApplicationRecord允许我们使用验证（validates，在2.3.2节里提过）来强制为某些属性添加限制。本节我们将学习几个最普通的验证：验证属性不能为空（NOT NULL）、长度、属性格式和唯一性。在6.3.2节，我们再添加最后一个验证。我们将在7.3节会看到当属性不满足条件时，验证会为我们提供一些方便调试的信息。

### 6.2.1 有效性测试
如注3.3里提到的，测试驱动开发并不是总是正确的，但是模型验证（Model validates）是TDD适用的完美场景。要确保validates实现了我们想要的功能，如果不通过测试来验证，我们是不会放心的。

我们的方法是从验证模型对象开始，把这个对象的属性设置为无效属性，然后测试它实际上是无效的。作为一道安全网，我们首先写一个测试来确保初始化模型对象是有效的。这样，当验证测试失败时我们就知道什么是真正的原因（而不是因为刚开始初始对象无效）。

为了让我们开始，代码清单6.1的命令产生了一个测试用户的初始化测试，尽管在这里，它是实际是空得（代码清单6.4）。

```
代码清单 6.4： The practically blank default User test.
# test/models/user_test.rb
 require 'test_helper'

 class UserTest < ActiveSupport::TestCase
   # test "the truth" do
   #   assert true
   # end
 end

```

为了为有效的对象写测试，我们先通过特殊的setup方法创建一个有效的User模型对象@user，（在第三章练习里讨论过），它会自动在每个测试运行前运行。因为@user是一个实例变量，它会自动在所有测试里都可以使用，我们通过“valid?”方法来测试它的有效性（6.1.3节）。如代码清单6.5显示的一样。

```
代码清单 6.5： 测试user是否有效。绿色
# test/models/user_test.rb
require 'test_helper'

class UserTest < ActiveSupport::TestCase

  def setup
    @user = User.new(name: "Example User", email: "user@example.com")
  end

  test "should be valid" do
    assert @user.valid?
  end
end
```
代码清单6.5使用普通的assert方法，在这里假如@user.valid?返回true，测试就成功，返回false意味着失败。

因为我们的User模型现在还没有添加任何validates，因此初始化的测试应该通过：
```
代码清单 6.6： 绿色
$ bundle exec rails test:models
```
这里我们使用rails test:models来只运行模型测试（和5.3.4节的rails test:integration比较）

### 6.2.2 非空验证
可能最基础的验证就是验证属性非空，它只是验证所给的属性非空。例如，本节我们确保name和email列在用户保存到数据库前是非空的。在7.3.3节，我们将看见怎样把这种要求求添加到新用户的注册表格。

我们将通过在代码清单6.5里的测试开始来测试@user对象的name属性非空。如同代码清单6.7所示，我们需要做的就是设置@user 的name属性为空白的字符串（在这个例子，一个空格字符串），然后检查（使用assert_not方法）User对象是无效的。

```
require 'test_helper'

class UserTest < ActiveSupport::TestCase

  def setup
    @user = User.new(name: "Example User", email: "user@example.com")
  end

   test "should be valid" do
    assert @user.valid?
  end

  test "name should be present" do
    @user.name = "    "
    assert_not @user.valid?
  end
end
```

到这里，模型测试应该是红色的：
```
代码清单 6.8： 红色
$ bundle exec rails test:models
```

正如我们在第二章练习中看到的一样，验证name属性存在的方法是使用validates方法，参数为presence: true，如在代码清单6.9里显示的一样。
presence: true参数是仅包含一个元素的可选哈希；回忆4.3.4节讲到在方法中如果最后的参数是哈希，大括号可以省略。（如5.1.1节提到的，在Rails里可选哈希是循环主题。）

```
代码清单 6.9： 验证name属性非空。绿色
# app/models/user.rb
 class User < ApplicationRecord
   validates :name, presence: true
 end
```

代码清单6.9可能看起来像魔术，但是validates其实是个普通的方法。使用圆括号的等价的形式如代码清单6.9所示：

```
  class User < ApplicationRecord
    validates(:name, presence: true)
  end
```

让我们顺便进入控制台来看看把验证加入到我们的User模型后的效果：

```
$ rails console --sandbox
>> user = User.new(name: "", email: "mhartl@example.com")
>> user.valid?
=> false
```
这里我们用valid？检查user变量的有效性，当对象没有通过一个或多个有效性验证就返回false，当所有的有效性验证都通过就返回true。在这个例子里，我们仅有一个有效性验证，所以我们很容易猜到那个测试没有通过，但是通过检查有效性验证失败后生成的errors对象可以让我们明确的知道违反了那条：
（错误信息暗示Rails使用blank?方法验证属性的存在，我们在4.4.3节看见过）

因为用户无效，企图把用户保存到数据库自动失败了：
```
>> user.save
=> false
```

因此，代码清单6.7的测试应该是绿色的：

```
代码清单 6.10： 绿色
$ bundle exec rails test:models
```

模仿代码清单6.7，写一个测试email属性非空的测试（代码清单6.11），应用程序代码应该可以通过（代码清单9.13）。

```
代码清单 6.11：验证email属性非空的测试。红色
# test/models/user_test.rb
require 'test_helper'

class UserTest < ActiveSupport::TestCase

  def setup
    @user = User.new(name: "Example User", email: "user@example.com")
  end

  test "should be valid" do
    assert @user.valid?
  end

  test "name should be present" do
    @user.name = ""
    assert_not @user.valid?
  end

  test "email should be present" do
    @user.email = "     "
    assert_not @user.valid?
  end
end
```

```
代码清单 6.12：验证email属性非空。 绿色
# app/models/user.rb
class User < ApplicationRecord
  validates :name,  presence: true
  validates :email, presence: true
end
```
到这里，属性非空的验证结束了，测试集应该是绿色的：
```ruby
代码清单 6.13： 绿色
$ bundle exec rails test
```

### 6.2.3 长度验证

我们已经限制每个用户都需要一个非空的名字，但是我们应该更进一步：用户名需要在Sample App上显示，所以我们应该在长度上再加一些限制。随着6.2.2节我们所做的工作，这一步很容易。

没有关于名字长度最长是多长的科学依据，我们只是以50作为合理的上限，这意味着字符长度为51的名字太长了。另外，尽管这不是问题，有的用户email地址可能超过字符串最大长度，对于许多数据库来说是255是合理的上限。在6.2.4节的格式验证不会强加这样一个限制，所以我们将在这节添加一个长度验证。代码清单6.14显示了测试运行的结果。

```ruby
代码清单 6.14：测试name长度的有效性。 红色
# test/models/user_test.rb
require 'test_helper'

class UserTest < ActiveSupport::TestCase

  def setup
    @user = User.new(name: "Example User", email: "user@example.com")
  end
  .
  .
  .
  test "name should not be too long" do
    @user.name = "a" * 51
    assert_not @user.valid?
  end

  test "email should not be too long" do
    @user.email = "a" * 244 + "@example.com"
    assert_not @user.valid?
  end
end
```
为了方便，我们使用"字符串相乘”在代码清单6.14里来创建51个字符的字符串。我们可以通过控制台来看看怎么工作的：

```ruby
>> "a" * 51
=> "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
>> ("a" * 51).length
=> 51
```
email长度验证创建字符太长的有效的email地址：

```ruby
>> "a" * 244 + "@example.com"
=> "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
aaaaaaaaaaa@example.com"
>> ("a" * 244 + "@example.com").length
=> 256
```

到这点，代码清单6.14应该是红色的：

```
代码清单 6.15： 红色
$ bundle exec rails test
```
为了让它们通过，我们需要使用验证参数来限制长度，就是length，和maximum参数一起强迫上边界（代码清单6.16）。

```ruby
代码清单 6.16：为name属性添加长度（length）验证。绿色
# app/models/user.rb
class User < ApplicationRecord
  validates :name,  presence: true, length: { maximum: 50 }
  validates :email, presence: true, length: { maximum: 255 }
end
```
现在我们的测试集再次通过，我们可以前往更具有挑战性验证：email格式验证。

### 6.2.3 格式化验证

我们对name属性的有效性验证仅仅有最低的限制--非空和不超过51个字符--然而email属性必须满足更严格的要求。到目前为止，我们仅仅拒绝空email地址和长度超过255个字符的email地址；本节我们将要求email地址遵从大家熟知的user@example.com模式。

无论是测试还是有效性验证都不能彻底的让我们接受有效的email，拒绝所有无效的email地址。我们将通过几个有效的和几个无效的email地址开始学习。为了创建这些集合，我们使用%w[]来创建字符串数组，这个技术很值得学习。我们通过控制台来学习一下：
```ruby
>> %w[foo bar baz]
=> ["foo", "bar", "baz"]
>> addresses = %w[USER@foo.COM THE_US-ER@foo.bar.org first.last@foo.jp]
=> ["USER@foo.COM", "THE_US-ER@foo.bar.org", "first.last@foo.jp"]
>> addresses.each do |address|
?>   puts address
>> end
USER@foo.COM
THE_US-ER@foo.bar.org
first.last@foo.jp
```
这里我们使用each方法（4.2.3节）遍历了addresses数组的元素。学会了这个，我们就可以写一些基础的email格式验证的测试。

因为email格式验证很麻烦而且很容易出错，我们将通过一些有效的email地址测试来在验证中捕捉可能出现的任何错误。换句话说，我们想要确保不只是无效的地址如“user@example,com”被拒绝，而且有效的地址，像“user@example.com”必须被接受。（现在，当然，因为所有非空email地址目前都是有效的。）有效email地址的代表性例子显示在代码清单6.18里。

```
代码清单 6.18： 测试有效的email地址。绿色
# test/models/user_test.rb
require 'test_helper'

class UserTest < ActiveSupport::TestCase

  def setup
    @user = User.new(name: "Example User", email: "user@example.com")
  end
  .
  .
  .
  test "email validation should accept valid addresses" do
    valid_addresses = %w[user@example.com USER@foo.COM A_US-ER@foo.bar.org
                         first.last@foo.jp alice+bob@baz.cn]
    valid_addresses.each do |valid_address|
      @user.email = valid_address
      assert @user.valid?, "#{valid_address.inspect} should be valid"
    end
  end
end

```

注意这次我们给assert传递了第二项可选的参数，用来在测试失败时输出提示信息。在这个例子中，可以通过它来辨别是那个地址引起测试失败：

```
assert @user.valid?, "#{valid_address.inspect} should be valid"
```
(这使用插值inspect方法，在4.3.3节提过）。在输出的调试信息中包含引起测试失败的地址在测试里尤其有用。否则无论是那个email地址引起测试失败只输出行号，这对于所有email地址来说都一样，对于识别问题的根源来说不够明显。

接下来我们将为无效的email地址添加有效性验证测试，例如“user@example,com”（句号替换成逗号），和user_at_foo.org(没有‘@’标志）。在代码清单6.18和6.19里包含一个自定义错误信息来输出引起测试失败的确切地址。
```ruby
代码清单 6.19：email格式有效性验证测试。 红色
# test/models/user_test.rb
require 'test_helper'

class UserTest < ActiveSupport::TestCase

  def setup
    @user = User.new(name: "Example User", email: "user@example.com")
  end
  .
  .
  .
  test "email validation should reject invalid addresses" do
    invalid_addresses = %w[user@example,com user_at_foo.org user.name@example.
                           foo@bar_baz.com foo@bar+baz.com]
    invalid_addresses.each do |invalid_address|
      @user.email = invalid_address
      assert_not @user.valid?, "#{invalid_address.inspect} should be invalid"
    end
  end
end
```

现在测试应该是红色的：
```
代码清单 6.20： 红色
$ bundle exec rails test
```
为验证email格式的应用程序代码使用format验证，它的形式是这样的：
```ruby
validates :email, format: { with: /<正则表达式>/ }
```
给定的正则表达式用来验证属性。正则表达式是非常强大的（难解的）匹配字符串的语言。这意味着我们需要构建正则表达式来匹配有效的email地址，不匹配无效的email地址。

实际上email官方标准有一个完整的匹配email地址的正则表达式，但是它太庞大了，而且非常晦涩，最后可能适得其反。本书中我们采用更具可编程特色的正则表达式，实践中证明这个表达式更加实用，即：
```ruby
VALID_EMAIL_REGEX = /\A[\w\d\-.]+@[a-z]\d\-.]+\.[a-z]+\z/i
```

为了理解它，表6.1把它分解成一小块一小块。

表达式 | 意思
----|----
/\A[\w\d\-.]+@[a-z]\d\-.]+\.[a-z]+\z/i | 完整的正则表达式
/ | 正则表达式开始
\A | 匹配字符串开始
[\w+\-.]+ | 起码一个单词、加号、短线或点
@ | 字符@
[a-z\d\-.]+ | 起码一个字母、数字、短线或者点
\. | 转义.
[a-z]+ | 起码一个字符
\z | 匹配字符串结尾
/ | 正则表达式结尾
i| 忽略大小写

表6.1： 分解有效email正则表达式

尽管你通过表6.1可以学到很多知识，要真正的理解正则表达式我认为使用交互的正则表达式匹配器如[Rubular](http://www.rubular.com/)是非常有必要
的（图6.7）。Rubular网站为创建正则表达式制作了一个漂亮的交互界面，还提供了很顺手的正则表达式规则参考。我鼓励你用浏览器打开Rubular学习表6.1的知识--眼过千遍不如手过一遍。（注意：假如你在Rubular中使用表6.1的正则表达式，我推荐不要加\A和\z字符，这样你就可以同时匹配几个email地址）

![图6.7：很棒的Rubular正则表达式编辑器](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/rubular.png)

将表6.1的正则表达式添加到email格式有效性验证，如代码清单6.21：

```ruby
代码清单 6.21：带正则表达式的email格式有效性验证。绿色
# app/models/user.rb
class User < ApplicationRecord
  validates :name,  presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX }
end
```

这里正则表达式VALID_EMAIL_REGEX是一个常量，在Ruby里常量用首字母大写的变量名来表示。代码：
```ruby
VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX }
```
确保仅仅和模式匹配的email地址才被当做是有效的。（上面的表达式有一个值得关注的弱点：它也接受一些无效的地址如包含连续的点，例如“foo@bar..com.”这类的email地址。解决这个问题需要更复杂的正则表达式，我把这个留下来作为作业（6.5节）。

到了这里，测试应该是绿色的：
```terminal
代码清单 6.22： 绿色
$ bundle exec rails test:models
```

这也意味着仅仅剩下一个限制性条件：要求email地址唯一。

### 6.2.5 唯一性验证

为了限制email地址唯一的性（以便我们能使用它们当做用户名），我们为validates方法传递:uniqueness参数。但是警告一下：这里有个重要的预告，所以不要只是掠过这节--最好仔细读读。

我们先通过一些简短的测试开始学习。在我们先前的模型测试中主要使用User.new，它只是在内存创建了一个Ruby对象。但是对唯一性测试来说，我们实际需要先在数据库中存储一个数据。唯一性验证的初始化email测试显示在代码清单6.23里。

```ruby
代码清单6.23：重复email的测试。红色
# test/models/user_test.rb

require 'test_helper'

class UserTest < ActiveSupport::TestCase

  def setup
    @user = User.new(name: "Example User", email: "user@example.com")
  end
  .
  .
  .
  test "email addresses should be unique" do
    duplicate_user = @user.dup
    @user.save
    assert_not duplicate_user.valid?
  end
end

```

这里使用的方法是通过@user.dup创建了一个和@user一模一样的用户，它们的email地址也一样。因为我们后来保存了@user，所以数据库里已经有了克隆的user的email地址，所以导致duplicate_user无效。

我们通过在代码清单6.23里email有效性验证中添加参数“uniqueness: true”，让新的测试通过，如代码清单6.24所示。

```ruby
代码清单 6.24：验证email地址的唯一性。 绿色
# app/models/user.rb
class User < ApplicationRecord
  validates :name,  presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: true
end
```

不过，我们还没完全结束。email地址通常是忽略大小写的，例如foo@bar.com和FOO@BAR.COM,
或者FoO@BAr.coM是一样的，所以我们的验证应该也纳入这个。测试忽略大小写是很重要的，我们在代码清单6.25里实现:

```ruby
代码清单 6.25：测试email唯一性对大小写不敏感。 红色
# test/models/user_test.rb
require 'test_helper'

class UserTest < ActiveSupport::TestCase

  def setup
    @user = User.new(name: "Example User", email: "user@example.com")
  end
  .
  .
  .
  test "email addresses should be unique" do
    duplicate_user = @user.dup
    duplicate_user.email = @user.email.upcase
    @user.save
    assert_not duplicate_user.valid?
  end
end
```

这里我们在字符串上使用upcase方法（4.3.2节提过）。这个测试和email唯一性验证测试刚开始时一样，但是用的是大写的email地址。假如这个测试看起
来有点抽象，让我们到控制台试试：

```ruby
$ rails console --sandbox
>> user = User.create(name: "Example User", email: "user@example.com")
>> user.email.upcase
=> "USER@EXAMPLE.COM"
>> duplicate_user = user.dup
>> duplicate_user.email = user.email.upcase
>> duplicate_user.valid?
=> true
```
当然，duplicate_user.valid?现在是true，因为唯一性验证是对字母的大小写敏感的，但是我们希望它的结果是false。幸运的是，:uniqueness接受一个
选项，:case_sensitive，仅仅为这个目的服务（代码清单6.26）。

```ruby
class User < ApplicationRecord
  validates :name,  presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false }
end
```

注意在代码清单6.26里，我们只是用case_sensitive: false替换了true。（Rails也推理uniqueness是true）。

到这里，我们的应用程序--随着重要的预告--已经实现了强制email属性唯一，我们的测试集应该通过：

```terminal
代码清单 6.27： 绿色
$ bundle exec rails test
```
不过还有个小问题，ApplicationRecord唯一性验证并不能保证数据库水平的唯一性。这里有一个场景，可以解释为什么：

1. Alice注册了示例应用网站，使用地址alice@wonderland.com
2. Alice不小心点了两次“提交”按钮，一下发送了两个请求
3. 接下来发生了：请求1在内存里创建了用户，通过了验证，请求2也一样，所以请求1保存了，请求2也保存了
4. 数据库中有了email地址重复的两条用户记录，尽管我们有唯一性验证。

上面的一系列场景听起来不合情理。相信我，这种情况非常可能出现：这种事情可以发生在任意一个流量较大的Rails网站（我曾经艰难的学会了）。幸运地是，解决方案是很简单的：我们需要强制数据库水平和模型水平一样唯一性。我们的方法是早email列创建一个数据库索引（注6.2），然后要求索引是唯一的。

    注6.2 数据库索引

    当为数据库表添加一列，考虑一下是否我们需要依靠这一列来查找记录是很重要的。例如，考虑一下在代码清单6.2里通过数据迁移创建的email属性。当我们允许用户登录到Sample App时（第七章），我们需要查找和用户提交的email地址相对应的用户记录。不幸地是，依靠幼稚的数据模型，通过email地址查找用户的唯一方法是浏览数据库里的每个用户，然后比较它的email属性和所给的email是否一致--这意味着我们可能不得不检查每一行记录（因为用户可能是数据库里最后一位）。在数据库业务里，这是著名的全表扫描，对于一个有着成千上万的用户的真实网站来说是一件[坏事](http://catb.org/jargon/html/B/Bad-Thing.html)。

    在email列上添加索引可以解决这个问题。为了理解数据库索引，我们可以想想书的索引，这对我们理解数据库索引是有帮助。在一本书中，为了查找所给的字符串，如“foobar”，你不得不扫描每页，纸版的全表扫描。假如我们换个方式，通过书的目录来查找，你可以通过在目录里查找“foobar”来找到所有包含“foobar”的页面。数据库索引和它的道理基本一样。

    为email列添加索引代表我们数据模型需求的更新，（如6.1.1节里讨论过的）在Rails中我们通过使用迁移来实现。在6.1.1节我们看到生成User模型时Rails为我们自动创建了一个新迁移（代码清单6.2）；在当前的例子，我们是添加一个结构到一个已经存在的模型，所以我们需要直接使用migration生成器来创建迁移：

```terminal
$ rails generate migration add_index_to_users_email
```

不像用户的迁移，email唯一性迁移没有预定义，所以我们需要把代码清单6.28里的内容填充进去。

```
代码清单 6.28：强制email唯一的数据迁移。
# db/migrate/[timestamp]_add_index_to_users_email.rb
class AddIndexToUsersEmail < ActiveRecord::Migration
  def change
    add_index :users, :email, unique: true
  end
end
```

这使用了Rails名为add_index的方法来在users表里的email列添加一个索引。索引本身不强制唯一，但是选项unique: true会实现强制索引唯一。

最后一步是迁移数据库：

```
$ bundle exec rails db:migrate
```
(假如运行该命令失败了，试试退出其他正在运行的沙盒控制台会话，因为它会锁住数据库，阻止数据迁移。）

到这里，测试集应该是红色的，因为fixture中的数据破坏了数据库索引的唯一性，fixture包含了为测试数据库准备的示例数据。user fixture是在代码清单6.1里自动生成的，如代码清单6.29所示，email地址不是唯一的。（它们也不是有效的，但是fixture数据不必通过验证。）

```ruby
代码清单 6.29：默认的user fixture。红色
# test/fixtures/users.yml
# Read about fixtures at http://api.rubyonrails.org/classes/ActiveRecord/
# FixtureSet.html

one:
  name: MyString
  email: MyString

two:
  name: MyString
  email: MyString
```

因为直到第八章我们才需要fixture，所以现在我们直接移除它们，留一个空的fixture文件（代码清单6.30）。

```ruby
代码清单 6.30：空的fixture文件。 绿色
# test/fixtures/users.yml
 # empty
```

要确保email地址唯一还有一件事情：一些数据库的索引区分大小写，例如数据库会认为字符串“Foo@ExAMPle.CoM”和“foo@example.com”是不同的索引。但是我们的应用程序认为这些地址是一样的。为了避免这种不兼容，我们将标准化email列，统一使用小写的email。因此在把数据保存到数据前我们需要把“Foo@ExAMPle.CoM”转换为“foo@example.com”。我们通过callback来实现这个功能。callback是一种在ApplicationRecord对象生命周期的某个点被唤醒的特殊函数。在当前例子中，这个点就是对象被保存到数据库前。代码显示在代码清单6.31。（这是我们的第一次实现。我们会在8.4节再次讨论这个主题，在那里我们会使用Rails偏好的办法来定义callback函数。）

```ruby
class User < ApplicationRecord
  before_save { self.email = email.downcase }
  validates :name,  presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false }
end
```

代码清单6.31里的代码是把块传递给before_save这个callback函数，然后使用downcase字符串方法把用户的email地址设置为小写。（为验证email小写写一个测试，这个任务当作练习（6.5节）留给你完成。）

在代码清单6.31里，我们可以通过如下方式为变量赋值，如
```ruby
self.email = self.email.downcase
```
(self指当前用户），但是在User模型里，右边的关键词self是可选的：

```ruby
self.eamil = email.downcase
```

我们在palindrome方法里reverse的上下文里简短地介绍过这个方法（4.4.2节），也要注意的左边的self在赋值时是不可选的，所以
```ruby
email = email.downcase
```
不会起作用。（我们将在8.4节更深入地讨论这个主题）

到这里，即便遇到上面提到的Alice的遭遇的场景程序也会很好的运行：数据库响应第一个请求，保存用户记录；当第二个请求到达时，因为它打破了数据库索引的唯一性限制，因此数据库拒绝保存它。（在Rails日志里会出现错误提示，但是不会对应用程序有任何伤害。）同时，在email属性上添加索引达成了我们的另一个目标：在6.1.4节里间接提到过的，如在注6.2节里说的，在email属性上的索引也解决了潜在的效率问题，当通过email地址查找用户时会避免对数据库进行全表扫描。

## 6.2 添加安全的密码

既然我们已经为name和email属性定义了有效性验证，我们以及准备好添加最后一个很基础的用户属性：安全的密码。实现的方法是要求每个用户拥有一个密码（还有密码确认)，然后在数据库里储存哈希版的密码。（这里可能有些读者会产生困惑。在当前的环境，哈希不是指4.3.3节中提到的Ruby的数据结构哈希，而是指应用不可逆的哈希函数对输入数据处理的结果。）我们也依据所给的密码进行用户验证。我们将在第八章通过用户验证允许用户登陆我们的网站。

为了验证用户名和用户提交的密码是否匹配，我们的方法是通过对用户提交的密码进行哈希运算，然后用我们得到的值和数据库里储存的值进行比较。假如它们两个相等则说明用户提交的密码是正确的，用户通过验证，否则要求用户重新输入密码。通过比较哈希值而不是直接比较原始密码的方式，我们就不需要保存用户的原始密码来验证用户。这意味着，即使我们的数据库被脱库（指数据库被黑客下载）我们网站的用户的密码仍然是安全的。

### 6.3.1 用哈希算法处理过的密码
大部分的密码安全机制都是通过Rails中的一个方法，has_secure_password，实现的。我们在User模型里添加如下代码：

```ruby
class User < ApplicationRecord
  .
  .
  .
  has_secure_password
end

```

在模型里包含这个方法会添加以下功能：
* 会把被哈希过的安全的password_digest添加到数据库
* 生成一对虚拟的属性（password和password_confirmation)，包含要求创建用户对象时要求它们一致的有效性验证
* 当密码正确时authenticate方法返回user对象，否则返回false

为了让has_secure_password正常工作，仅要求相应的模型中有一个名为password_digest的(密码摘要)属性。（digest是来自于cryptographic
哈希函数的术语。这里，哈希过的密码和密码digest是一样的。）在User模型的例子中，生成如图6.8显示的数据模型。

![图6.8：带password_digest属性的数据模型](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_model_password_digest_3rd_edition.png)

为了实现图6.8的数据模型，我们需要为user模型添加password_digest属性。我们先生成一个数据迁移来实现这个目标。我们可以为数据迁移文件起任何我们想要的名称，但是用to_users结尾会方便一些。因为Rails通过识别它会自动构建为users表添加列的数据迁移。因此我们为数据迁移文件起名为add_password_digest_to_users,显示如下：
```ruby
$ rails generate migration add_password_digest_to_users password_digest:string
```

这里我们也提供了参数password_digest:string，我们想要添加的属性的名称和类型。（比较这个和代码清单6.1里生成的创建users表的数据迁移，我们为Rails提供了足够的信息来创建整个数据迁移，如代码清单6.32所示：

```ruby
代码清单 6.32：为用户表添加password_digest列的数据迁移。
# db/migrate/[timestamp]_add_password_digest_to_users.rb
class AddPasswordDigestToUsers < ActiveRecord::Migration
  def change
    add_column :users, :password_digest, :string
  end
end
```

代码清单6.32使用add_column方法来添加password_digest列到users表。应用它，我们将迁移数据库：为了创建密码摘要，has_secure_password使用最先进技术的哈希函数叫做[bcrypt](http://en.wikipedia.org/wiki/Bcrypt)。通过用bcrypt哈希密码，我们确保攻击者不会登进网站，及时它们拥有一份数据库备份。为了在示例网站使用bcrypt，我们需要把bcrypt gem添加到Gemfile（代码清单6.33）。

```
代码清单 6.33： Adding bcrypt to the Gemfile.
source 'https://rubygems.org'

gem 'rails',                '5.0.0'
gem 'bcrypt',               '3.1.7'
.
.
.
```
然后运行bundle install，如往常一样：
```
$ bundle install
```

### 6.3.2 安全密码（has secure password)

既然我们已经为User模型添加了has_secure_password方法要求的password_digest属性并且安装了bcrypt Gem，我们现在把has_secure_password添加到User模型，如代码清单6.34所示：

```ruby
代码清单 6.34：为User模型添加has_secure_password。 红色
# app/models/user.rb
class User < ApplicationRecord
  before_save { self.email = email.downcase }
  validates :name, presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false }
  has_secure_password
end
```
如在代码清单6.34说明的，红色表示现在测试失败。你可以通过命令行确认

```terminal
代码清单 6.35： 红色
$ bundle exec rails test
```

原因是，如在6.3.1节里提到的，has_secure_password要求虚拟属性password和password_confirmation的值非空并且一致。但是在代码清单6.25里的测试创建的@user变量还没有这些属性：

```ruby
def setup
  @user = User.new(name: "Example User", email: "user@example.com")
end
```
所以，为了再次让测试集通过，我们需要添加password和password_confirmation，如代码清单6.36显示的。
```
代码清单 6.36：添加password和password_confirmation。 绿色
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
end
```

现在我们看到了添加has_secure_password的用处（6.3.4节）了，但是现在我们还需要添加密码长度的限制。

### 6.3.3 密码长度

一般来说限制密码的最小长度可以让密码更难破译，这对程序设计来说是好的实践方法。在Rails里有许多[强制密码强度](http://lmgtfy.com/?q=rails+enforce+password+strength)的选择，但是简单起见，我们只是通过限制密码的最短长度和要求密码非空来加强我们的密码强度。要求密码不能少于6为是合理的，密码长度有效性验证测试如代码清单6.38所示：

```ruby
代码清单 6.38： 测试密码的最小长度。 红色
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

  test "password should be present (nonblank)" do
    @user.password = @user.password_confirmation = " " * 6
    assert_not @user.valid?
  emd

  test "password should have a minimum length" do
    @user.password = @user.password_confirmation = 'a' * 5
    assert_not @user.valid?
  end
end
```

注意在代码清单6.38里同时赋值的使用：
```ruby
@user.password = @user.password_confirmation = 'a' * 5
```
这条语句同时给password和password_confirmation赋值，（在这个例子，字符串长度为5，使用和代码清单6.14一样的乘法构建）。

你可能猜出来通过使用和maximum相应的minimum长度限制来验证用户名的长度的代码（代码清单6.16）：
```ruby
validates :password, length: { minimum: 6 }
```

把这个和presence验证组合（6.2.2节）来阻止空密码，User模型如代码清单6.39所示。（它证明了has_secure_password方法虽然包含存在验证，但是它仅适用于新纪录，当更新用户信息时会有问题。）
```ruby
代码清单 6.39： 完整的安全的密码实现。绿色
# app/models/user.rb
class User < ApplicationRecord
  before_save { self.email = email.downcase }
  validates :name, presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false }
  has_secure_password
  validates :password, presence: true, length: { minimum: 6 }
end
```

在这里，测试应该是绿色的：
```
代码清单 6.40： 绿色
$ bundle exec rails test:models
```

### 6.3.4 用户注册和验证

既然基本的用户模型创建完毕，我们现在开始在数据库里添加一个用户，这样我们就可以着手创建显示用户信息的页面（7.1节）；我们也将通过实例来验证一下在User模型中添加了has_secure_password方法后的效果，包括authenticate方法的作用。

因为用户还不能通过网页来注册Sample App--这是第七章的目标--我们先通过Rails控制台来手动创建一个新用户。简单起见，我们将使用在6.1.3节讨论过的create方法。不过我们现在不启用沙盒了，这样用户就可以被保存到数据库中。也就是说我们开启一个普通的rails console会话，然后用有效的name和email，有效的password和password_confirmation来创建一个用户：

```ruby
$ rails console
>> User.create(name: "Michael Hartl", email: "mhartl@example.com",
?>             password: "foobar", password_confirmation: "foobar")
=> #<User id: 1, name: "Michael Hartl", email: "mhartl@example.com",
created_at: "2014-09-11 14：26：42", updated_at: "2014-09-11 14：26：42",
password_digest: "$2a$10$sLcMI2f8VglgirzjSJOln.Fv9NdLMbqmR4rdTWIXY1G...">
```

为了检查上面的命令确实在数据库中添加了一条新用户记录，让我们用SQLite数据库浏览器来看看开发数据库里的users表，如图6.9所示。（假如你正使用云IDE，你应该先下载数据库文件图6.5）。注意在图6.8里的各列对应了数据模型相应的属性。

![图6.9：在SQLite数据库db/developement.sqlite3里的用户行](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/sqlite_user_row_with_password_3rd_edition.png)

返回控制台，我们能从代码清单6.39里看到has_secure_password的效果，即password_digest属性的值：

```ruby
>> user = User.find_by(email: "mhartl@example.com")
>> user.password_digest
=> "$2a$10$YmQTuuDNOszvu5yi7auOC.F4G//FGhyQSWCpghqRWQWITUYlG3XVy"
```

它的值就是我们新建的用户的哈希版的密码（“foobar”）。因为它是通过bcrypt gem构建，因此想使用摘要来还原密码估计是不太现实的。

如6.3.1节提到的，has_secue_password自动添加为相应的数据模型添加了authenticate方法。这个方法决定是否所给的密码对某个特定的对象（这里是指用户对象）是有效的，它是通过计算这个对象密码的摘要和数据库里的password_digest结果是否一致。通过我们刚创建的用户我们可以先试试几个无效的密码：

```ruby
>> user.authenticate("not_the_right_password")
false
>> user.authenticate("foobaz")
false
```
这里user.authenticate对无效的密码返回false。假如我们使用正确的密码，authenticate会返回用户自身：
```
>> user.authenticate("foobar")
=> #<User id: 1, name: "Michael Hartl", email: "mhartl@example.com",
created_at: "2014-07-25 02：58：28", updated_at: "2014-07-25 02：58：28",
password_digest: "$2a$10$YmQTuuDNOszvu5yi7auOC.F4G//FGhyQSWCpghqRWQW...">
```

在第八章，我们将使用authenticate方法来验证用户登陆。实际上，它也证明authenticate是否返回用户本身并不重要，我们主要关心的是它返回了逻辑值为true的值。因为用户对象不是nil或false，所以它也符合我们的要求：

```ruby
>> !!user.authenticate("foobar")
=> true
```
## 6.4 结语

在这章我们从零开始，创建了一个包含name、email和password属性的用户（User）模型，并添加了几个重要的强制有效的验证规则。另外我们现在也有了安全地通过密码验证用户的方法。这里我们仅仅通过十二行代码就取得了大量的功能。

在下一章，第七章，我们将为新用户创建一个注册表和显示用户信息的页面。在第八章，我们使用6.3节的验证机制来让用户登陆网站。

假如你正使用Git，假如你还没有提交，现在是提交的好时候：

```ruby
$ bundle exec rails test
$ git add -A
$ git commit -m "Make a basic User model (including secure passwords)"
```

然后合并进主分支，推送至远程仓库：

```terminal
$ git checkout master
$ git merge modeling-users
$ git push
```

为了让用户模型在生产环境也可以工作，我们需要先在Heroku上运行数据库迁移，我们通过heroku run完成：

```
$ bundle exec rails test
$ git push heroku
$ heroku run rails db:migrate
```

我们可以通过生产环境里的控制台来确认这个工作：
```ruby
$ heroku run console --sandbox
>> User.create(name: "Michael Hartl", email: "michael@example.com",
?>             password: "foobar", password_confirmation: "foobar")
=> #<User id: 1, name: "Michael Hartl", email: "michael@example.com",
created_at: "2014-08-29 03：27：50", updated_at: "2014-08-29 03：27：50",
password_digest: "$2a$10$IViF0Q5j3hsEVgHgrrKH3uDou86Ka2lEPz8zkwQopwj...">
```

### 6.4.1 这章我们学到了什么
* 数据迁移允许我们修改应用程序的数据模型
* ApplicationRecord有创建和操作数据模型的大量方法
* ApplicationRecord validates方法允许我们在模型里放置数据限制规则
* 普通的validates包括非空、长度和格式
* 正则表达式是晦涩的，但是很强大
* 当允许数据库水平的属性值唯一，定义数据库索引提高了查询效率
* 我们可以使用内建的has_secure_password方法为模型添加安全的密码

## 6.5 练习

1. 从代码清单6.31里添加一个email小写化测试，如代码清单6.41所示。这个测试使用reload方法为了从数据库里加载一个方法和assert_equal方法。为了确认代码清单6.41测试了正确的需求，先注释before_save行，让它变红，然后去掉注释让它变绿。

2. 通过运行测试集，确认before_save回叫能使用“bang”方法email.downcase!来直接修改email属性，如代码清单6.42所示。
3. 如在6.2.4节提到的，在代码清单6.21里的email正则表达式允许在域名里有连续的点的无效email地址，例如“foo@bar..com”格式的地址。把这个地址加入到代码清单6.19里来得到失败测试，然后使用在代码清单6.43里更复杂的正则表达式来让测试通过。

```ruby
代码清单 6.41：以代码清单6.31为基础，添加email小写化测试。
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
  test "email addresses should be unique" do
    duplicate_user = @user.dup
    duplicate_user.email = @user.email.upcase
    @user.save
    assert_not duplicate_user.valid?
  end

  test "email addresses should be saved as lower-case" do
    mixed_case_email = "Foo@ExAMPle.CoM"
    @user.email = mixed_case_email
    @user.save
    assert_equal mixed_case_email.downcase, @user.reload.email
  end

  test "password should have a minimum length" do
    @user.password = @user.password_confirmation = "a" * 5
    assert_not @user.valid?
  end
end
```

```ruby
代码清单 6.42：before_save回叫函数的另一种实现方法。绿色
# app/models/user.rb
 class User < ApplicationRecord
  before_save { email.downcase! }
  validates :name, presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false }
  has_secure_password
  validates :password, presence: true, length: { minimum: 6 }
end
```

```ruby
代码清单 6.43：不允许域名中包括两个及以上连续的点。绿色
# app/models/user.rb
class User < ApplicationRecord
  before_save { email.downcase! }
  validates :name, presence: true, length: { maximum: 50 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-]+(\.[a-z\d\-]+)*\.[a-z]+\z/i
  validates :email, presence:   true,
                    format:     { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false }
  has_secure_password
  validates :password, presence: true, length: { minimum: 6 }
end
```
