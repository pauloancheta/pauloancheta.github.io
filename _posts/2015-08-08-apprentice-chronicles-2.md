---
layout: post
title: "Apprentice Chronicles: 2"
description: "The second week as a Brewhouse Apprentice"
categories:
  - cucumber
  - git
  - bash
  - design
  - agile
date: 2015-08-08 08:00
---

## killing postgres ##
As I was testing with cucumber, I have tried to cancel the test because it was running way too long. The problem is when I have cancelled all the running task, my postgres db stopped working. I consulted with one of my mentors and he wrote a brilliant script that kills all waiting postgress tasks which is really interesting

<!-- break -->

{% highlight ruby %}
# I saved this in ~/helper_scripts/clean-pg.rb
pss = ` ps aux | grep postgres | grep waiting`
if pss
  pss.each_line do |ps|
    next if ps.include?('grep')
    pid = ps.split(/\s+/)[1]
    system "kill #{pid}"
  end
  puts "----------------------------- All waiting postgress killed"
end
{% endhighlight %}

I do however made it output a message so I can bind it to bash.

{% highlight bash %}
# ~/.bash_profile
# HELPER SCRIPTS
alias hammertime='ruby ~/helper_scripts/clean-pg.rb'
{% endhighlight %}

Now, everytime I type `hammertime` all waiting process will be killed!

## git-fu ##
Probably the most useful git command I have learned is cherry picking. It did sound very complicated at first but in practice, it is not that hard. It is especially useful using it when working with a lot of branches in an agile environment. While working on my branch, I had to get a commit from another branch. I could just copy the code from that commit but it would cause merge conflicts but one command rules it all.

{% highlight bash %}
# while on your branch
git cherry-pick [commit SHA from another branch]
{% endhighlight %}

## importance of design ##
I tried to build my own start up but I have struggled so much on a lot of things. Building a good product is important, but building a product that the user will use and love is more important and that is where design comes in. During and after bootcamp, I thought design is all about making everything pretty but design as I am taught now is more than just CSS.

In designing a product, it is important to understand what the product is trying to solve. As for my start-up, the main problem I was trying to solve is to make it easier for people to choose dishes from a menu by eliminating the dishes that they cannot consume. What I didn't ask my self is how the problem should be solved: how would I get the information of the Restaurants? Who would maintain the databse of the restaurant menu? How would a user input their allergies and ensure that their personal information would not be compromised?

The take away in design is it is not just a matter of creating a beautiful page but it is also designing the user experience which is more complex than what I though of it initially.

## ready, aim, fire! target: _blank ##
Usually when I write a link in rails, I never think about firing a new window because I never really needed to but there are cases where a new window is better. Sometimes, you don't really want them to leave your page and a modal is not an option. Add `target="_blank"` to a link to make it open to a new window.
