---
layout: post
title: "The Seed, The Task, And The Service Object"
date: 2022-04-23
categories:
  - ruby
  - service objects
description: >
  Some
  multiline
  description
header-img: "img/posts/img.jpg"
---

It's been a long time since I've written something about software because I found myself thinking
that I have to preface this post with a postulate: "There are multiple ways of writing code". In my
last post over 4 years ago, I've written that software evolves, just like our language. What I would
write now, may not be the way we do things in the future. In fact, the way I prefer to write my own
code may not necessarily be the way you or your company may write code, because we have our own
rhetoric. Many may disagree with how I write my code, but for those who don't have a preference may
find a hidden gem (pun intended).

I've been in a company that adopted the
[Spotify squad](https://www.atlassian.com/agile/agile-at-scale/spotify) model whilst having a Rails
majestic monolith. This creates a very interesting problem wherein some teams would structure their
code to depend on a particular data to be present, while some teams don't necessarily care about
that (or at least not yet). The solution that our team came up with, is better seeding. But as most
developers know, what we write now, may not necessarily be correct tomorrow. The question now is how
do we maintain a seed that changes constantly?

## Seed

Seeds are found in `db/seed.rb` and can be executed by running `rails db:seed`. What we write in the
seed will run as if it was pasted to the rails console. It also runs as part of `rails db:setup`,
where it creates the database, runs the migration, and running our seed script. It's a vary handy
tool when onboarding a new developer. Now let's establish ground rules for seeds:

1. Seeds never run in production
While we can be carefule not to change particular data, and we can write it to be idempotent, the
chances of having a bug that may affect production data is pretty high. We cannot write tests,
there's very limited visibility on the executed commands, and there is no rolling back.

2. Seeds should never call an outside API
Seeds should create a dumb initial data that will help you set your environment with enough data to
work on. We don't need the seed sending email notifications to anyone. Nor do we want it to create
payments to Stripe. Having an outside API on a seed will just create unnecessary complexity and
failure points.

```ruby
# db/seeds.rb

# create Company
company = FactoryBot.create(:company)

# create roles
admin_role = FactoryBot.create(:role, :admin)

# create users
admin = FactoryBot.create(:user, company: company, role: admin_role)
user_1 = FactoryBot.create(user, company: company, role: :editor)
user_2 = FactoryBot.create(user, company: company, role: :viewer)

# create default posts
default_title = "Default Post"
admin.posts.create!(title: default_title)
user_1.posts.create!(title: default_title)
```

In this example, I'm using a gem called [FactoryBot](https://github.com/thoughtbot/factory_bot). It
is a very powerful tool that allows developers to create random data. It is also possible to use
plain old ruby syntax in here. But in cases where FactoryBot is already used in tests anyway, it
might make sense to just use it.

But this doesn't solve our problems. Given a big team with each creating their own seed, it would
force a lot of developers to clear their database all the time. Of course we can make the seed
idempotent with methods like `#first_or_create`, but then that would mean we have to know what we
want right away, which is not possible all the time.

## Tasks




## ServiceObject
These objects should be dumb. If we follow the SOLID principle, these objects do one thing. Most
rails application that I've seen use it to create resources that are dependent from each other.
While this could be used for seeding, ServiceObjects would be better fitted to create resource or
one resource with all its dependents. For example:

```ruby
company = Company!(name: "My Company")
user = User.create!(name: "Paulo", email: "paulo@example.com", company: company)
user.send_email_notification
```

If for example our user needs a default `post`, one way we can achieve this is by creating
`after_create` actions, which could become unweildy in the future. Tests would start become slower,
it would be painful to write tests just because we have a side effect to a model. My recommendation
is to use ServiceObjects instead so that they can be tested in isolation and creating a state of the
world would be done through the objects

```ruby
company = Company!(name: "My Company")
User::Create.call(company: company, name: "Paulo", email: "paulo@example.com")

class User::Create
  def self.call(...)
    u = User.create!(...)
    u.posts.create(title: "Default Post")
    u.send_email_notification
  end
end
```


```ruby
# create default company
company = FactoryBot.create(:company, name: "Test Company 1")
admin = FactoryBot.create(:user, company: company, role: :admin, name: "Admin", email: "admin@example.com")
user_1 = FactoryBot.create(:user, company: company, role: :editor, name: "User", email: "user_1@example.com")
user_2 = FactoryBot.create(:user, company: company, role: :viewer, name: "User", email: "user_1@example.com")

# create company with no editors or viewers
company = FactoryBot.create(:company, name: "Lonely Company")
admin = FactoryBot.create(:user, company: company, role: :admin, name: "Admin", email: "admin@example.com")

# create company with multiple admin
...
# create company with editors no viewers
...

# create default posts
...
```

## Task
When a service needs to run more than once, it is useful to create a `rake task` for them. Tasks are
ideally report it's own progress as well. For example

```ruby
task :send_email_notification do
  user_count = User.count
  User.find_each(batch_size: 100).with_index do |u, i|
    print "#{i+1}/#{user_count} Sending email notification to #{u.id} "
    u.send_email!
    puts "DONE!"
    NotificationService.call("Sent message to #{u}")
  end
end
```

### Cron
One of the patterns I've seen that is really useful handling different `rake tasks` that rely on
cron to run, is that it is namespaced through their increments:

```ruby
namespace :monthly do
  task :send_email_notification do
    user_count = User.count
    User.find_each(batch_size: 100).with_index do |u, i|
      ...
    end
  end
end

namespace :daily do
  ...
end
```

With this, it is more intuitive debugging different tasks running on different times.

### Migrations
In most cases, migrations are written in `db/migrate/` folder. These migrations are inheriting from
`ActiveRecord::Migration` and have a specific DSL. The best practice for this folder is that they
don't use any class inside the migration because by doing so, we are coupling these immutable files
to a class that should be easy to change or delete. Which is why, when migrating data, it is best
to do it as a rake task.



## Tying them all together

