---
layout: post
title: "Brewhouse Apprenticeship: 1"
date: 2015-08-03 09:46:00
catergories: 
  - ruby
  - I18n
  - cron
  - "data objects"
description: "Week 1 of my awesome apprenticeship"
---

## @stuff.pluralize() vs I18n ##
On my first day, my task is very simple. It is to take a word, `person` and pluralize it if there are more. My solution was not terrible because it works but it suddenly became controversial. My solution was the write a ternary operator to solve it. 

{% highlight ruby %}
# file.erb
<%= @collection.size > 1 ? 'people' : 'person' %>
{% endhighlight %}

<!-- break -->

That wasn't that bad, I thought. It works anyway so thats ok. The code is not too long that it would be ugly to look at inside a paragraph. But there are better solutions according to my mentors. Although this works, how about making it shorter? They have suggested a way better solution using either the `pluralize` method or `I18n` which stands for internationalization. So I researched on both and see what I like best.

I've used `.pluralize` before so I thought that if I use this, my code would just become longer. But I was only thinking of just one use case of that method. The method can also take in an integer as a parameter and pluralizes it according to the integer. So my code could be written as:

{% highlight ruby %}
# file.erb
<%= "person".pluralize(@collection.size) %>
{% endhighlight %}

Although it didn't change much, I do agree that this is more readable and easier to understand. But lets say for example that the sentence is constructed a bit differently: "1 person bought potatoes". Then the code would be a lot more!

{% highlight ruby %}
# file.erb
<%= @collection.size %> <%= "person".pluralize(@collection.size) %> bought potatoes
{% endhighlight %}

Then in this case, I think `I18n` would be really useful. Consider this:

{% highlight yaml %}
# config/locales/en.yaml
en:
  new_person:
    one: '1 person'
    other: '<%= count %> people'
{% endhighlight %}

{% highlight ruby %}
# file.erb
<%= I18n.t("new_person", count: @collection.size %> bought potatoes
{% endhighlight %}

Even though there is more code necessary for this, I do think it is more maintanable and has more benefits. I did choose the I18n just because there is less duplication of code even though I think pluralize is more verbose.

## Unleash the CRON jobs! ##
`Rake tasks` is an unknown ground for me. I have never scheduled a task before. I have seen the `whenever` gem before but I haven't used it in a project yet (or at least not successfully). I do remember fiddling with it on my CodeCore Bootcamp but it sadly remained a branch on my repository and was never merged. But after completing my first task, I'm feeling more confident and ready to take it on!

Although it could be done manually from `rake` task and modifying heroku, `whenever gem` provides an easy way to set up cron jobs. [Rake task tutorial from www.ultrasaurus.com](http://www.ultrasaurus.com/2009/12/creating-a-custom-rake-task/)

After reading the `whenever` documentation, I think it was pretty straight forward. Specify the time in UTC, and specify which file to run.

{% highlight ruby %}
# config/schedule.rb
every 1.day, at: "12:00 pm" do
  runner "Model.method"
end
{% endhighlight %}

And in my case, I ran a service object. It is easier to create services and test that service that putting it in a model. One of my mentors wrote a really good article on service objects and why you should start using it if you are not using it yet: [Be nice to others and your future self, use data objects](http://brewhouse.io/2015/07/31/be-nice-to-others-and-your-future-self-use-data-objects.html)

## refactor until it becomes stupid simple ##
Refactoring is an important task. There will come a time where it would be really hard to add or modify features because the code knows too much about one thing. Having separate data objects makes changes a lot simpler. This is quite hard for me to grasp at first since I have only started coding full time this year. An important lesson that I learned though is that if a code knows too much about something, changes in the future will be a lot more complex. Consider this:

{% highlight ruby %}
# app/services/vegetarian.rb
class Vegetarian
  ...
  def is_vegetarian?
    Dish.recipe.ingredients.where(category: 'vegetarian').size == Dish.recipe.ingredients.size
  end
end
{% endhighlight %}

Although now it is really clear to me now that I should create methods in `Dish` class to get the associations, it wansn't very obvious for me before. My goal before is just to make it work and never thought about the future changes. Yes, it works but the most important part is the future changes. Every app/program changes over time, this is inevitable. If you make the future changes easier and refactor it, then the future changes would be easier to write and would have a less probability of bugs.

{% highlight ruby %}
# app/services/file.rb
class Vegetarian
  ...
  def is_vegetarian?
    Dish.vegetarian_ingredients.size == Dish.ingredients_list.size
  end
end

# app/models/dish.rb
class Dish 
  def ingredients_list
    Dish.recipe.ingredients
  end
  
  def vegetarian_ingredients
    ingredients_list.where(category: 'vegetarian')
  end 
end
{% endhighlight %}

I do understand that this is a bad example since `is_vegetarian?` should go to class `Dish`, but I'm sure you get the idea. The service object should not know too much about the class itself. By doing it this way, the code is less prone to bugs and much simpler to edit in the future.

## Bang it twice! ##
One of the most useful methods methods that I have seen as well is the double bang for a method that is supposed to return true or false. Consider a method that returns a string or a `nil`. Having a string is truthy, and nil is falsy; but what about an empty string? Sometimes you want it to be falsy as well, and double bang makes that happen.
{% highlight ruby %}
Class Dish
  def description_exists?
    !!description
  end
end
{% endhighlight %}
