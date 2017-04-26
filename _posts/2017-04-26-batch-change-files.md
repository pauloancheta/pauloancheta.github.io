---
layout: post
title: "Batch Change Files"
date: 2016-04-22
categories:
  - base64-images
  - jsonapi-resources
  - rspec_api_documentation
description: "Fast and simple. But more importantly, it's fast"
---

Renaming strings is a job that we as programmers can never avoid. The only thing we can do is either make it fun or make it fast that we don't have to think about it anymore. So, I tried to do both. Following what I have learned from the book [Sed & Awk](https://www.amazon.com/Sed-Awk-Dale-Dougherty/dp/1565922255), I wrote my simple script. Just a small note, I am using [The Silver Searcher](https://github.com/ggreer/the_silver_searcher) here. And yes, it is faster than [Ack](https://beyondgrep.com/).

{% highlight bash %}
#!/bin/zsh
# run chmod 777 on this file

# use silver searcher to find all matches and use -l option to print only the filenames
# convert file names into an array so we can loop through it
files_array=("${(@f)$(ag -l 'OLD_STRING')}")
for file in $files_array
do
  # edit file and substitute OLD_STRING to NEW_STRING
  sed -i '' -e 's/OLD_STRING/NEW_STRING/g' $file
done
{% endhighlight %}
