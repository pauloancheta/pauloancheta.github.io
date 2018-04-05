---
layout: post
title: "Understanding a Ruby WAT"
date: 2018-03-23
categories:
  - ruby
  - wat
description: "When ruby returns something that you don't understand, you write a blogpost about it"
header-img: "img/posts/wat.jpg"
---

In my last post, I really tried to write about devops since that is what I'm currently doing
professionally.  But, it just doesn't tickle my brain. If you, my freakin awesome reader have read
the latest [StackOverflow Survey](https://insights.stackoverflow.com/survey/2018/?utm_source=Iterable&utm_medium=email&utm_campaign=dev-survey-2018-promotion),
devops is [one of the highest paid positions](https://insights.stackoverflow.com/survey/2018/?utm_source=Iterable&utm_medium=email&utm_campaign=dev-survey-2018-promotion#work-salary-by-developer-type).
Judging from that I should be writing more about devops, but I just don't think it is my passion
at the moment. But Ruby is. Or at least for now. So, here's what I found in Ruby in the past few weeks

> If you haven't seen [WAT](https://archive.org/details/wat_destroyallsoftware), I really think you should watch it first before reading on.

## Let's talk about Ruby

Few weeks ago, a co-worker found something really interesting. He posted on our slack channel this
line of code. Does anyone know what the output of this is?

{% highlight ruby %}
"(data)".gsub("(data)", "15\\01\\2018")
{% endhighlight %}

(If you were too lazy to open your irb here's the output: `"15(data)1018"`)
<iframe src="http://gifimage.net/wp-content/uploads/2017/06/wat-gif-10.gif"
  width="500" height="300" frameBorder="0" allowFullScreen>
</iframe>


## Mansplaining the WAT

Well, honestly, the real WATs are the code that we just don't understand. In Gary Bernhardt's example
of the ruby WAT is more like a ruby worst practice. If you're careful, you won't really get to see it.

In a pretty long slack thread, one co-worker found out what was going on in that line of code.
He just exclaimed, "AH! That is a regex backreference!".

<iframe src="https://giphy.com/embed/rmi45iyhIPuRG" width="356" height="480" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/yes-score-rmi45iyhIPuRG">via GIPHY</a></p>

Cool, interesting, but, I didn't really understand what he meant.

After a quick research, I did remember that there was a [Ruby Tapas](https://www.rubytapas.com/) for
this. I know that I have seen an episode where Avdi briefly touched on ruby backreference using
gsub. (If you are not subscribed to his tapas, you should be ashamed. Subscribe now and watch
[this video](https://www.rubytapas.com/2015/01/19/episode-274-backreference/)).

Let's do a few experiments. From the tapas, we now know that `\\0` is a back reference. My first
theory of `\\0` was that it is the first capture group. But according to the tapas, `\\1` is the
first capture group.

{% highlight ruby %}
"hello world".gsub("hello", "\\0 mundo")
# => "hello mundo world"
{% endhighlight %}

<img src="http://i0.kym-cdn.com/photos/images/original/000/173/580/Wat.jpg" width="500" height="300" frameBorder="0">

Let's do another experiment. This time, let's use one word with more indicators

{% highlight ruby %}
"hello".gsub("hello", "[0]: \\0 [1]: \\1")
# => "[0]: hello [1]:"

"hello".gsub(/(\w)(\w)/, "[0]: \\0 [1]: \\1")
# => "[0]: he [1]: h[0]: ll [1]: lo"
{% endhighlight %}

Well from that, we can infer that `\\1` (1st capture group), is the first letter that matches the
first parenthesis in the regex, or the first match. `\\0` on the other hand is anything that matches
the whole regex.

The method `gsub` iterates over the whole string, trying to search for matches, hence having two
`[0]` capture groups and two `[1]` capture groups.

If we look at the last output closer, the last `[1]` capture group captured two characters instead
of one. That is because gsub returned the rest of the chars from the capture group.

## Back to the first problem

Let's try the example again but without the back references:
{% highlight ruby %}
"(data)".gsub("(data)", "15 01 2018")
  ^             ^           ^
  |             |           |
data            |           |
            matcher         |
                        replace data with this
# => "15 01 2018"
{% endhighlight %}

Now using back references:
{% highlight ruby %}
"(data)".gsub("(data)", "15\\01\\2018")
                            ^   ^
                            |   |
                            | does not match anything
                    replace this with what was captured
# => "15(data)1018"
{% endhighlight %}

In Ruby Conference Australia few weeks ago, [Katie McLaughlin](https://twitter.com/glasnt) gave a
talk on WAT's and it has inspired me to dig deep and find out for myself why WAT's happen. (Her
slides are in [here](https://github.com/glasnt/talks/tree/gh-pages/2018_03_RubyConfAU/).) It is not
because the language is bad; it is usually a part of the language that we don't understand.
Understanding is the key.

---

If you liked this blog post, or if you think I can make this better, I would love to hear from you!
Send a tweet or an email; whatever suits your fancy.
