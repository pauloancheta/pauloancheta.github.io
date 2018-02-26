---
layout: post
title: "Scalable Rails Application using AWS and Kubernetes Part 1"
date: 2017-12-27
categories:
  - docker
  - rails
  - postgres
  - nginx
description: "Introduction to Docker"
header-img: "img/post-bg-03.jpg"
---

Most devs who learn how to code, learn by deploying to [Heroku](https://www.heroku.com).
Which by all means a really good place to start. I've used Heroku for most of my personal projects
and I have never found a reason to move out of it. However, most enterprise companies are not set
up with heroku; most often they choose [AWS](https://aws.amazon.com) for a variety of reasons. Some
would choose AWS/ rolling their own deployment because of security reasons. With AWS IAM roles,
all instances can be configured into a specific access. EC2 Auto-scaling group, servers can be
configured to scale up and scale down within a trigger.

All that is neat but we can go beyond what AWS can offer by using [Kubernetes](https://kubernetes.io/).
Kubernetes is a project from Google. It is an open source system for automating deployments,
scaling and managing containers.

## Introduction to Docker

In order for us to deploy our rails application, we will first have to containerize our application
using [Docker](https://www.docker.com/). By using docker, we can pull a docker "image" and use that
to run our rails application. Here's the final `Dockerfile` that should be on the parent directory
of your application.

{% highlight docker %}
FROM ruby:2.4.2

RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs

WORKDIR /hello_container

ADD Gemfile ./Gemfile
ADD Gemfile.lock ./Gemfile.lock
RUN bundle install

ADD . .

EXPOSE 3000

CMD ["bundle", "exec", "rails", "server", "-b", "0.0.0.0"]
{% endhighlight %}

Now that we have the file we need, let's dive into the code and understand line by line on I am doing
here. Think of docker as a layered cake. The base of the cake doesn't change much and the layer on top
are open for variations. You can still remove the bottom of the cake and change the bottom layer
completely, but that is always a tedious process.

<iframe src="https://giphy.com/embed/vtmWF8WdeqIKY"
  width="480" height="392" frameBorder="0" class="giphy-embed" allowFullScreen>
</iframe>
<p><a href="https://giphy.com/gifs/food-colors-cake-vtmWF8WdeqIKY">via GIPHY</a></p>

## Understanding Layers

The idea is that your layers are reusable. You can reorder how your cake is layered but there is
always a preferred order. Think of the `FROM` statement as the base cardboard where the layers go
on top of. A `FROM` directive is almost always first. It can only be preceded by an `ARG`, but we
don't need to concern ourselves of that at the moment.  In the Dockerfile example, `FROM` dictates
that we are going to install `ruby:2.4.2`. This is called the "base image" of our Dockerfile.

The `RUN` directive runs a bash or shell script. In this example, it runs `apt-get` to our base
image. All it does here is making sure to install 3 libraries in our file `build-essential`,
`libpq-dev`, and `nodejs`. `build-essential` installs packages required debian packages, `libpq-dev`
installs postgresql, and `nodejs` installs node.

The next directice creates a directory, and sets it as the working directory. This is accomplished
with `WORKDIR`.

Here comes an important bit. We then Add the `Gemfile` and `Gemfile.lock`. The reason why we're
adding these files first, is that so we can run `bundle install` and not bundle again every time we
change something. Ideally, we only run bundle install if `Gemfile` and `Gemfile.lock` changes again.

Next we add all the other files to the current directory. Everytime we change the other files with
the exception of the Gemfiles, we are only changing this layer which should make it really easy for
us to compile.

## Testing

We can test our `Dockerfile` by running `docker build . -t hello-container`. This tells docker, "Hey docker,
I want to build this current directory and tag it with `hello-container`". Docker then would run all
directives one by one and register a layer for each.

To run it, we can use `docker run -it --rm hello-container`. This should run the server of rails
server.

---

I've covered enough in here for today. In my next post, I'll write a brief introduction to K8s!
