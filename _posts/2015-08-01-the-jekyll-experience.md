---
layout: post
title:  "From Plain HTML to Jekyll!"
date: 2015-08-01 14:50:00
categories: firstPost
description: "My first Jekyll post"
---

For a long time, I have been wishing to learn how to convert my personal site to Jekyll - and now I have finally gotten to it. I intend to write posts regarding my Brewhouse apprenticeship on this blog, but for now let me just review how my first Jekyll experience went.

Coming from Ruby on Rails, Jekyll was a struggle at first. There was no concept of model-view-controller (MVC); no routes to look at, the documentation doesn’t tell you how the system actually works. (Or I guess I just didn’t read the documentation that well.) But eventually I figured it out after a few hours.

<!-- break -->

## The Jekyll Gem ##

The first thing to know about jekyll is that it is a gem. So, you do have to install it with `gem install jekyll` which is pretty straight forward. The first hiccup that I got from it was converting my old site into Jekyll. At first, I thought there was a shortcut into it. I started running `jekyll new blog` onto my old website folder and tried pushing it to github. And of course, as most github pages users know, this approach doesn’t work (but if does, please let me know!). With that failure, I then made my actual first blog using Jekyll.

## Post? What the... ##
The second struggle that I got was how the hell does layouts get posted? It was so easy on ruby since with Ruby on Rails, you just have to write this to display all the posts:

{% highlight ruby %}
<% @posts.each do |post| %>
  <p><%= post.content %></p>
<% end %>
{% endhighlight %}

That is pretty straight forward. Looping through all the posts in the database. Assuming that you have `@posts = Post.all` in the controller that renders your view.

In Jekyll though, it was confusing at first since I received a bunch of errors. Since there is actually no controllers, the first question I had after looking at the `index.html` was: How does the app know what goes in `post.date` ? Or how about if I put `post.something_else`? How can I do that?

While looking at the markdown file from the `_posts` folder, you would see the there is a YAML part on top. This is where the post attributes are. Just imagine a loop of posts where a post can have anything you specify in this part.

{% highlight yaml %}
# _posts/yyyy-mm-dd-title.md
---
layout: post
title: "My super awesome title"
categoriges: potato bacon "You can also put a string with spaces"
description: "my super awesome description"
random_variable: "my randon variable"
---
{% endhighlight %}

So looking at this, and the documentation, `layout` is what the markdown would use as a ‘layout’ - pretty straight forward. However, everything else can be accessed through `post`. Treat it as a JSON object. Knowing this, I deduced that it uses the post layout, while `index.html` uses default layout. post.date is the date you have put in the YAML part of the markdown, and `{{ "{{ content" }}}}` is whatever is inherited by that layout. In default layout, it can be the `post` `layout` or the `page` layout. In the `post` layout, it is whatever is in the body of the markdown file.

A few more questions: How does it know which CSS files to render? And how do I add more links in the nav bar?

## Adding Files ##

In the default layout, there is a line that includes the head, header and footer html. This reminds me of partials on Ruby on Rails, and it helps to keep the layout very neat and organized. Jekyll provides a really good way of adding CSS files since it supports SASS. There is one CSS folder that has the `main.scss` which imports files from the `_sass` folder.

{% highlight scss %}
// css/main.scss
@import
  "base",
  "layout",
  "syntax-highlighting",

// just add your own partials here!
  "my-own-sass-file"
;
{% endhighlight %}

Adding links to the navbar is really easy too. Just create a new folder and add the markdown inside it. As long as you follow the front-matter (The yaml part on top of the markdown), then it should work.

## Conclusion ##

Learning Jekyll is not as hard as I thought it would be. There were a few hiccups but nothing overly complicated since the documentation is really good. I did look at other jekyll layouts to know more about how to customize my page and I am pleased with what I am able to make.
