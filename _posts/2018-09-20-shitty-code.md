---
layout: post
title: "This Code is Sh*t"
date: 2018-09-20
categories:
  - ruby
  - clean code
description: >
  It's always easy to diss a code that you don't have a context on.
  But things change when you start asking why it happened in the first place.
header-img: "img/posts/agenda.jpg"
---

Coming from a consultancy advocating for TDD (Test-Driven-Design) and clean code, it's easy to look
at one method or class and say "this is pretty horrible; look at all the complexity of this method.
I could squish it down to 2 lines at most!” which leads to saying, "wtf"s and "omg"s which greatly
reduces the team morale because of the number of facepalms per minute per developer. However, I've
been on both sides of the spectrum. I have written methods and classes that have been a headache to
fix for some people.

## Is Bad Code Actually Bad?
The short answer is yes. There is no excuse for a badly named method or an obscure class that does
not tell any history of what happened. But developers are writers. We write code for other developers to
read and understand. It is useless to write a one liner if no one understands it, but in most cases,
we forget one thing that is part of every spoken language, rhetoric.

Oxford dictionary defines rhetoric as:
> "1The art of effective or persuasive speaking or writing, especially the exploitation of figures
> of speech and other compositional techniques."

It does not matter if you can write the most verbose method or class, because if it is not
effectively trying to communicate to the reader, context will be lost, and it will still be a "WTF"
for the reader.

{% highlight ruby %}
# create a payment when user enough
# money to pay for the item
if wallet.has_money? && wallet.money > item.price
  wallet.charge!(item.price)
end

# move the logic into a smaller method
if wallet.has_enough_for?(item.price)
  wallet.charge!(item.price)
end

# Further abstraction to a service object
UserPayment.charge!(item_price: item.price)
{% endhighlight %}

But writing effectively in software development is harder, much harder.

Software development is a collaborative effort. It can be done with one person but in most cases,
projects are co-authored by more than 2 people. And with every group of people, rhetoric changes.
Effective communication and writing differs per language, per team and per company. To make it more
difficult, employees come and go. Writing style changes and projects start to evolve. We have a name
for this change, it is called legacy code.

## Legacy Code Is Not Bad
I personally believe that no individual should be blamed for badly written code. Writing software is
a collaborative effort, peer reviews get rid of the badly written code. A member of the team should
read the code and make sure that it's readable, elegant and free from possible typographical error.
And since badly written code gets fixed during review, how then do we still have facepalm moments?
I believe it is because of rhetoric.

<div style="width:100%;height:0;padding-bottom:56%;position:relative;"><iframe src="https://giphy.com/embed/jICAI3lhoypRHHMV1i" width="100%" height="100%" style="position:absolute" frameBorder="0" class="giphy-embed" allowFullScreen></iframe></div><p><a href="https://giphy.com/gifs/Team-Penske-jICAI3lhoypRHHMV1i">via GIPHY</a></p>

Our language changes, it evolves, it is alive. In the last century, people did not use the word
"cool" to describe something that has astounded them. "Awesome" was reserved for something that
truly takes their breath away, though we use these words sparingly in today's rhetoric.

Software practices change, it evolves, it's alive. Most languages that we use today gets new commits
almost everyday. New improvements happen every day. The new hip way to write `Service Objects` 2
years ago, will change in the next few years if it hasn't because software rhetoric is alive and
changing. Yesterday microservices was the new hot thing, and having monoliths were shunned upon; but now, I'm not so sure about that.

## If You Can't Beat Them, Join Them
We cannot simply update everything that has been written to have the new rhetoric. In writing bug
fixes or adding any change, consider the past, consider the change, and consider future readers
because pushing a fix is never enough. When introducing new rhetoric, be aware of the change of
writing. Some legacy code could be avoided by simply understanding how other methods or classes
have been written. It is always easier to rewrite a class but it is almost always not the best
solution because we lose history, and it adds the potential to introduce new bugs.

And before you judge someone's code, consider the amount of bugs they had to go through before
reaching that ugly solution. Good code looks good because it has not gone through the test of time.
