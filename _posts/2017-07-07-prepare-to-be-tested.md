---
layout: post
title: "If You Don't Test, Prepare To Be Tested"
date: 2017-07-07
categories:
  - ruby
  - tdd
description: "Create a habit before throwing it away"
header-img: "img/post-bg-03.jpg"
---

Yes, I know Test-Driven-Development(TDD) is dead, but before throwing away this great habit because
all the great programmers said so, please remember where you were, what helped you get there and
what would you advice for developers who are starting out.

Remember those times when you just added your code just to make it work and you are completely fine
with a very cryptically named method? And the operations that you added is so convoluted you can't blink because
you would lose your train of thought while reading it? And to top it off, you think your code looks
so damn good because no one can understand it, and that makes you feel so smart? Okay, so maybe you
were not that bad, but you know someone else who writes code like that? The funny thing is, that
person, is a potential contributor to your repository. It's always not you who would be reviewing
it and someone could just totally make your easily readable, understandable and untested code be
the nightmare class you have on your service objects right now.

The fact is, when you started out this class, it was easy to understand. It was less than a hundred
lines long and it can totally just slide out without being tested because it's so succinct and verbose.
But classes don't stay to be small, succinct and verbose. Bugs creep in and other people will add methods to it.
So small that most people would say, "yeah sure, it's a one-line change". And then another person comes in
with a small feature request and adds one more line. A few sprints after, it remains untested but doubled
its size.

100% test coverage is often unnecessary, I agree. Some classes doesn't need to be tested. But the biggest
benefit to a TDD, in my opinion, is that it teaches good coding habits. It trains us to know what
we are writing first before we even write it. It creates that pause to make us think and understand
the task at hand.

As an example, let's create a simple service object Chemex that will brew our coffee so we can try testing
its filters out of it.

{% highlight ruby %}
class Chemex
end

Rspec.describe Chemex do
  it "works" do
    expect(1).to eq 1
  end
end
{% endhighlight%}

I usually start with this template. I create my class with this template so I know that my test works.
Once it passes, then I start thinking, what should my class do. I have a vague idea of what it does so I
will add the test first to describe what I want it to do.

{% highlight ruby %}
it "accepts a filter with coffee in it" do
  expect {
    Chemex.new(filter: Filter.new() ).call
  }.to raise_error CoffeeNotFoundError
end
{% endhighlight %}

Notice that this error will just raise an error left right and center. It would raise an error because
my `Chemex` doesn't support arguments at the moment, and it hasn't implemented the method `call`. But
with it, it ensures 2 things. That proper variables are set before calling it, which leads me to think that
I should have the supporting classes, and a proper error, which lets me to know what is missing right away
should I call this class again and forget to pass in a `Filter` without coffee in it.

{% highlight ruby %}
class CoffeeNotFoundError < StandardError; end

Class Filter
  def initialize(with_coffee: false)
  end
end

class Chemex
  def initialize(filter: filter)
    raise CoffeeNotFoundError unless filter.with_coffee
  end

  def call
    # it works
  end
end

Rspec.describe Chemex do
  it "accepts a filter with coffee in it" do
    ...
  end
end
{% endhighlight %}

