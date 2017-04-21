---
layout: post
title: "Concurrent Coffee"
date: 2017-04-21
categories:
  - concurrency
description: "Learning concurrency through coffee"
---

A student at a developer bootcamp where I used to work as a teaching assistant asked me what the difference was between concurrency and parallelism. It was a daunting task since I don't know if I can give a good answer to. And since I am not confident enough to give him a good answer, I gave a [video](https://www.youtube.com/watch?v=cN_DpYBzKso) for him to watch. It is a video of Rob Pike explaining how concurrency is different from parallelism. Although I think it will answer his question, I do feel that I should be able to answer the question on my own if I was to say that I know the concept of parallelism and concurency.

Few days after the event, I had a conversation with my fiancée about how they work at Starbucks. There I had my "aha!" moment. I could try explaining concurrency and parallelism through brewing coffee!

First, let's understand what process is. According to Merriam-Webster, a process is "a series of actions or operations conducing to an end". Brewing a single cup of coffee is a process. We put a paper filter on top of the chemex brewer, we pour hot water around the inside of the paper filter and discard the water. We then add the ground coffee to the filter and slowly pour the hot water in. Then we discard the paper filter and pour the coffee to the cup. These steps is a single process.

![Single Chemex](/img/concurrent_coffee/single-chemex.jpg)

Merriam-Webster describes concurence as: "the simultaneous occurrence of events or circumstances". When we pour hot water in to the kettle, it is simultaneously brewing coffee. So if we have 2 chemexes, we are concurrenly brewing coffee. It is not the chemex that is concurrent, but the kettle itself is concurrent. The 2 chemex, however, is not concurrent; it is parallel.

![Double Chemex](/img/concurrent_coffee/double-chemex.jpg)
