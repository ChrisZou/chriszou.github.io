---
layout: post
title: Rails Authentication From Scratch
date: 2016-09-15T17:53:02+08:00
comments: true
post_id: rails-authentication-from-scratch
---

这是一个对railscast250的学习笔记，所有copyright belongs to Railscast作者 Ryan Bates

### 1. 创建User Model

`rails g resource user email password_digest`

这里有两点说明，一是关于rails g resource，你可能对rails g scaffold熟悉，那resource是什么呢？在这里，scaffold和resource都是rails generator，就是可以帮你自动生成很多琐碎的东西的助手，这样你就不用自己一遍一遍写类似的东西了。resource跟scaffold最主要的不同是，resource不会帮你生成对应的controller里面的任何个action以及他们对应的任何view；第二点需要说明的是“password_digest”这个字段，这个字段比较关键，因为接下来我们要用到rails为我们提供的has_secure_password这个helper method，这个method默认需要用到这个字段。

### 2. Migrate database
在命令行执行`rake db:migrate`创建你的database

### 3. 打开model里面的user.rb,加上 has_secure_password

```ruby
class User < ActiveRecord::Base
    has_secure_password
end
```
`has_secure_password`这个方法我们前面提到过，它的作用是，利用前面提到的`password_digest`字段，给你提供`password` 和`password_confirmation`两个字段，并帮你做一些常见的validation，比如：密码不能为空，长度必须小于72字符，把password和password_confirmation两个字段进行对比，确保他们相等，等等。

### 4. 安装bcrypt gem
前面提到的has_secure_password这个方法需要用到一个gem叫bcrypt，这个gem在你创建project的时候默认已经加到你的Gemfile里面去了，只不过默认被注释掉了。我们打开Gemfile，找到那一行并uncomment掉。接着执行
`bundle install`
进行安装，安装完这个gem以后，重启rails server让它生效。

### 5. 添加extra validation
我们可以在User model里面添加额外的validation，比如email不能为空，也必须唯一：

```ruby
class User < ActiveRecord::Base
  validates_presence_of :email
  validates_uniqueness_of :email

  has_secure_password
end
```
你也可以根据自己的需要，再加其他的validation，比如邮箱格式，密码最小长度等等。

### 6. 添加controller代码，用于创建用户。

打开app/controllers/users_controller.rb, 添加new 和create action

```ruby
class UsersController < ApplicationController
  def new
    @user = User.new
  end

  def create
    @user = User.new(user_params)
    if @user.save
      redirect_to root_url
    else
      render new
    end
  end

  private
  def user_params
    params.require(:user).permit(:email, :password, :password_confirmation)
  end
end
```
这些都是rails 标准的new，create action写法，没什么好说的。

### 7. 添加new view
在app/views/users 文件夹下面新建一个文件 new.html.erb, 添加以下代码：

```ruby
<h1>Sign Up</h1>
<%= form_for @user do |f| %>
<% if @user.errors.any? %>
<div class="error_messages">
<h2>Form is invalid</h2>
<ul>
<% @user.errors.full_messages.each do |message| %>
<li><%= message %></li>
<% end %>
</ul>
</div>
<% end %>

<div class="field">
<%= f.label :email %><br />
<%= f.text_field :email %>
</div>
<div class="field">
<%= f.label :password %><br />
<%= f.password_field :password %>
</div>
<div class="field">
     <%= f.label :password_confirmation %><br />
     <%= f.password_field :password_confirmation %>
</div>
<div class="actions"><%= f.submit "Sign Up" %></div>
<% end %>
```
</div>
这个也是标准的new action写法，没什么好说的。

### 8. 添加注册的链接到页面上。

正常情况下，如果用户没有登录，你很可能希望在你网站的每一个页面都显示注册、登录的链接，所以，在这里一个好的做法是将链接加到application layout里面。具体加到什么位置，什么样的style由你觉得，这里我简单地将“sign up”链接加到右上角，共参考。
打开app/views/layouts/application.html.erb, 在body后面加上

```ruby
<div class="user_navigation">
     <%= link_to ’Sign up', new_user_path %>
</div>
```
接下来设置style,让链接靠右对齐就好了。打开app/assets/stylesheets/application.css，加入以下内容：

```css
.user_navigation {
     width: 100%;
     text-alignment: right;
}
```
这样，打开任何一个网页，点击右上角的login，就可以跳转到注册用户的界面了(http://yourhost/users/new)。如下图所示：

到这个页面输入邮件，密码，确认密码，就可以成功创建一个用户了。

到这里，关于用户注册的部分我们基本就做好了，但是还没有做任何关于用户登录的事情，接下来我们来做这部分工作。

用户登录：
### 9. generate 一个新的controller来负责登陆这部分逻辑。通常这个controller名字叫sessions controller, 执行：
rails g controller sessions new
这样就创建了一个sessions controller，同时还给了他一个new action。

### 10. 修改routes.rb
通过上面的命令，rails 自动帮你在routes.rb里面加了一条记录：
get 'sessions/new'
但对于我们来说，一个session除了new，还要有create，destroy这些操作，所以session对于我们来说更像是一种resource，所以我们把routes.rb里面的那条记录删了，改成resource形式的routes，在routes.rb里面添加一行代码：
resources sessions
这样，我们就有了session对应的标准的resouces形式的routes。
在这里也做一个小小的解释，在标准的rails action中，new的作用只是呈现一个表单给用户填写（在这里，是一个创建session，即登陆，的表单），而create的作用才是正真创建这个session的过程（在这里就是执行登陆流程）。

### 11. 创建new session的view。
打开app/views/sessions/new.html.erb, (这个文件在generate controller的时候就已经帮你创建了，不用自己新建）加入以下内容：

```
<h1>Log In</h1>

<%= form_tag sessions_path do %>
<div class="field">
<%= label_tag :email %><br />
<%= text_field_tag :email, params[:email] %>
</div>
<div class="field">
<%= label_tag :password %><br />
<%= password_field_tag :password %>
</div>
<div class="actions"><%= submit_tag "Log In" %></div>
<% end %>
```
这也是一个简单的登录form，没什么好说的。
现在打开http://localhost:3000/sessions/new ，是不是可以显示一个login的表单了？

### 12. 在上面的表单中，点击登陆，页面将执行post 到sessions controller的create action，所以接下来我们要再create action 里面写执行登陆过程的代码。
打开app/controllers/sessions_controller.rb, 添加以下代码：

```ruby
def create
     user = User.find_by_email(params[:email])
     if user && user.authenticate(params[:password])
          session[:user_id] = user.id
          redirect_to root_url, notice: 'Logged in!'
     else
          flash.now.alert = 'Email or password is invalid'
          render 'new'
     end
end
```

这里稍微解释一下上面的代码，第二行User.find_by_email(params[:email]) 根据email查找用户，如果存在这个email的用户，则返回这个用户，否则返回nil，如果用户不为nil，则执行user.authenticate(params[password])  这个方法，用来验证密码是不是正确。这个方法也是前面提到的has_secure_password这个方法为我们创建的。如果密码正确，则把user id保存到session里面，这里的session是一个类似于hash的对象，这是rails为我们提供的session机制。

### 13. 在页面上加上登陆的链接。
跟前面添加注册的链接类似，这里在application layout里面简单加上登陆的链接：

```html
<div class="user_navigation">
     <%= link_to ’Sign up', new_user_path %> or <%= link_to 'Login', new_session_path %>
</div>
```
现在，打开任何一个页面，在右上角都有“Sign up”,和”Login “的连接了，点击login，则会跳转到login 的页面。在那个页面输入email和密码，就可以进行登录了。登陆完以后，浏览器会跳转到root页面。
到这里，基本的用户注册和登陆流程就完成了。
但是这里还有个小问题，你登陆完了以后，发现右上角还有“sign up”和“login”的链接，这个不是很合理，所以接下来我们完善一下这一点。

### 14. 要让页面根据相应的登陆状态显示相应地内容，首先我们要有一个地方可以获取用户是否登录这个信息，由于这个信息一般来说是所有controller共用的，所以一个好的做法是将这个东西放到ApplicationController里面，打开app/controllers/application_controller.rb，在ApplicationController类里面加入以下代码：
     def current_user
          @current_user ||= User.find(session[:user_id]) if session[:user_id]
     end
因为我们自己创建的controller都是继承自ApplicationController，所以这样的话，所有的controller都可以调用current_user这个方法来判断当前有没有用户登录了。
然而这里还有个问题，在controller里面，我们已经可以判断用户登录与否了，但是在view里面，我们无法调用到这个current_user方法，要在view里面知道有没有用户登录，我们可以写一个helper method，这里有一个很方面的方法是在ApplicationController里面加上一行：
helper_method :current_user
这样下来，current_user 不仅仅是一个controller里面的方法，也是一个helper method，可以在view里面调用了。

### 15. 接下来我们重写application layout关于sign up和login这两个链接的部分，利用current_user helper method获取用户登录状态，然后分别显示对应的内容。打开app/views/layouts/application.html.erb, 将前面写的
<div class="user_navigation">
     <%= link_to 'Sign up', new_user_path %> or <%= link_to 'Login', new_session_path %>
</div>
换成以下的代码：
<div class="user_navigation">
     <% if current_user %>
          Logged in as <%= current_user.email %>
     <% else %>
          <%= link_to 'Sign up', new_user_path %> or <%= link_to 'Login', new_session_path %>
     <% end %>
</div>
现在，用户登录后右上角将显示用户的email地址，然后未登陆的时候右上角将显示Sign up和Login的链接。

### 16. 添加logout的链接。
我们继续完善这部分的流程，前面我们已经能够正确显示用户是否登录的状态了，但我们还需要给用户提供一个logout的链接，让用户可以logout，而不用等session自动过期。
用户logout的过程其实就是让session destroy的过程，也就是说，我们要在sessions controller里面添加一个destroy 方法。打开app/controllers/sessions_controller.rb，在SessionsController类里面加入以下代码：
def destroy
     session[:user_id] = nil
     redirect_to root_url, notice: 'You have logged out!'
end
很简单，对吧，让让用户退出登录其实就是清掉session里面的user_id就可以了。
接下来，我们给页面上添加一个logout链接，打开app/views/layouts/application.html.erb，在刚刚我们修改的那段代码里面加上一个link，如下：
<div class="user_navigation">
     <% if current_user %>
          Logged in as <%= current_user.email %>
          <%= link_to 'Logout', session_path('current'), method: :delete %>
     <% else %>
          <%= link_to 'Sign up', new_user_path %> or <%= link_to 'Login', new_session_path %>
     <% end %>
</div>
这里简单解释一下，因为我们要triggler sessions controller的destroy action，所以我们声明请求的方法为delete，这也是rails的标准routing方式。另外，triggler sessions controller的destroy action需要我们提供一个id参数，我们通过sessions_path方法的第一个参数传入这个id，这里我传入的是’current'，但其实你回去看看destroy action的实现方法，并没有用到这个id，所以这里传入任何值都是无所谓的。

现在，登陆以后，就可以在email右边显示一个logout链接了。
