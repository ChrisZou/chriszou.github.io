---
layout: post
title: "Use FactoryGirl in Rails"
date: 2016-02-29T06:57:43+08:00
comments: true
post_id: use-factory-girl-in-rails
---

[FactoryGirl](https://github.com/thoughtbot/factory_girl) is a gem aimed at replacing Fixtures. If you wonder that, take a look at [what's wrong with fixtures](http://www.dan-manges.com/blog/38).

### 1. Add `factory_girl_rails` to your Gemfile

```
group :test, :development do
    gem 'factory_girl_rails'
end
```

### 2. Add to your spec/rails_helper config block:

```
RSpec.configure do |config|
  #  Other config code
  config.include FactoryGirl::Syntax::Methods  # Add this line
end
```

### 3. Define factory

```
FactoryGirl.define do
	factory :user do
		username ‘foo’
		password ‘secret’
		email {“#{username}@example.com”} # define value dynamically, pass a block
	end
end
```

### 4. Use the factory in your spec

```
user = create(:user)
user = create(:user, username: ‘xiaochuang’, password: ‘xiaochuangchuang’)  # Override the default value.
user = build(:user)  # Will not save to database, faster than create
```

### 5. tips and tricks

#### 1. use sequence in model field
Say your user model has a validation of uniqueness for the attribute `username`, then if you call `create(:user)` multiple times without passing in the `username`, you’ll get an error. In this case, you can use `sequence` to auto generate username for you.

```
factory :user do
	sequence(:username) {|n| “user#{n}”}
end
```

Now everytime you `create(:user)` will return a user with a different username in the form: user0, user1, user2...

#### 2. Use associations
Assume you have a Post model that references to a User.

```
factory :user do
	username ‘xiaochuang’
	password ‘xiaochuangchuang’
end

factory :post do
	title ‘A new post’
	user #Yes, it’s just that simple.
end
```

However, the above usage assume that the user field is name `user`, if not, you can use `association` to customize the behavior.
`association :author, factory: :user, username: ‘xiaochuang’`

#### 3. Nested factory
Say your user model have an attribute called `admin`, this is used frequently and you don’t want to pass in the admin attribute very time. In this case, you can use a nested factory:

```
factory :user do
	username ‘xiaochuang’
	password ‘xiaochuangchuang’
	admin false

	factory :admin do
		admin true
	end
end
```

In your spec, you can just `create(:admin)` to create a user that set the `admin` to `true`.

#### 4. Use build instead of create if possible.
By using `create(:model)`, you create the model instance and save if to the database. In many cases if you don’t have to save it the the database, you can use `build(:model)`. This will build an instance but wouldn’t save to the database, which makes it a little bit faster.   
And also, in many cases you if you can just `Model.new`, you can bypass factories at all, which is even faster.
