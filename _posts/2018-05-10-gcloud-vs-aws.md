---
layout: post
title: "4 Reasons Why I Think Google Cloud Will Take Over Cloud Computing Space"
date: 2018-05-10
categories:
  - aws
  - google cloud platform
  - ruby
description: >
  I have been using AWS for a while now, and I've only tried using Google Cloud for a few days.
  Google, I'm impressed
header-img: "img/posts/arm_wrestling.jpg"
---

Few weeks ago, if you asked me, "which cloud services platform should I use?".  I would've said
heroku hands down, but if you have more time and you would want more flexibility, consider using
AWS (Amazon Web Services). With all the services AWS releases and how fast they are developing, it
seems like an obvious choice. Hands down you should use AWS S3 to store all your objects.
Cloudformation makes it really easy to create and tear down AWS services. AWS has dominated the
market for years. The support for their services are also pretty outstanding. The community usually
has answers for most of your needs. But in the past few days, I've been exploring the alternative:
Google Cloud -- and here's what I've seen so far.

## A better Client API
Back when I was studying at [CodeCore Bootcamp](https://codecore.ca), they have taught us how to use
AWS S3 which stands for "Simple Storage Service". Using their ruby client, it is really easy to
store files and retrieve files.

{% highlight ruby %}
require 'aws'
client = AWS::S3::Client.new
response = client.list_buckets

response.buckets.first
# => {:name=>"paulos-bucket-of-fun", :creation_date=>"2017-08-18T17:30:09.000Z"}
{% endhighlight %}

Each bucket is a file storage, filenames should be globally unique, and each files inside that
bucket is called an object. But the problem is, while it seems pretty straight forward, it does not
feel like ruby because of its conventions.

{% highlight ruby %}
client = AWS::S3::Client.new
response = client.list_buckets
# {:buckets => [{:name="paulos-bucket-of-fun, :creation_date=>"2017-08-18T17:30:09.000Z"}]}

response.class
# => AWS::Core::Response < Object
# A response object is a hash-like object that contains `{buckets: collection}`

my_bucket = response.buckets.first
# => {:name=>"paulos-bucket-of-fun", :creation_date=>"2017-08-18T17:30:09.000Z"}

# Now, what other methods can I use to query my bucket?
my_bucket.methods
# => ArgumentError: wrong number of arguments (given 2, expected 1)
{% endhighlight %}

<iframe src="https://giphy.com/embed/QajHhLKW3VRcs" width="480" height="318" frameBorder="0" class="giphy-embed" allowFullScreen>
</iframe><p><a href="https://giphy.com/gifs/QajHhLKW3VRcs">via GIPHY</a></p>
Oh no, the `#methods` has been overridden! Let's see how this compares to Google Cloud gem

{% highlight ruby %}
require "google/cloud/storage"

storage = Google::Cloud::Storage.new(project_id: 'paulos-example-project-id')
# => #<Google::Cloud::Storage::Project:0x007faae1898aa8

storage.buckets.class
# => Google::Cloud::Storage::Bucket::List < #<Class:0x007faae290c9c0>
# YAY! It returns an actual list

storage.buckets.first.methods
# ... A whole range of methods available to me
{% endhighlight %}

In my opinion, the Google Cloud api is much more straight forward, easy to use and conforms with the
conventions of ruby. They did not need to return a response object. It returns what I expect an
object to return. The naming sends a clearer message to a point that I can pretty much guess what
the method name is (`buckets.first.files` is much clearer to me than the aws equivalent
`s3.list_objects_v2(bucket: bucket_name)`).

## IAM
<iframe src="https://giphy.com/embed/26BREWCvD66KImhc4" width="480" height="360" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>
<p><a href="https://giphy.com/gifs/majorkey-future-jay-z-dj-khaled-26BREWCvD66KImhc4">via GIPHY</a></p>
While this is a VERY broad topic, I particularly like Google Cloud Platform IAM because it is tied
to my google account. This means, I don't have to create another account in another platform. While
it may seem that this might pose a risk for security, Google has made it clear on how you should
handle IAM for services. There's also no way (or at least I haven't found any) to embed my own
credentials inside a service. This means, if I want one my services to access a certain part of
my microservices or Google Cloud, I'll have to generate another type of access. Credentials are json
files. There are p12 files as well but that's the old way, we should be using the new hipster way.

{% highlight json %}
{
  "type": "service_account",
  "project_id": "project_id",
  "private_key_id": "some hash",
  "private_key": "-----BEGIN PRIVATE KEY-----\n-----END PRIVATE KEY-----\n",
  "client_email": "service-account@project_id.iam.gserviceaccount.com",
  "client_id": "123",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://accounts.google.com/o/oauth2/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/service-account%40project_id.iam.gserviceaccount.com"
}
{% endhighlight %}

From this file, I know that I am using a service account. I know that if I off-board one of the
developers, all our apps will be up and none of them will have any problems, all tests will still
work and it will still be a good day.

AWS IAM looks the same for an account owned by a person and by a machine. All we need to use is an
`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` and it should work. There will be no trace of what
kind of account uses these credentials and it is like trying to find a needle in a haystack just to
trace it back to the owner. The problem with this is that when the owner leaves the company, one of
the service just stops working. All this can be prevented by implementing good practices, but as
long as there is still a way for us to do it, mistakes can still happen.

## Monitoring
While AWS has really extensive monitoring, to make them consumable, I prefer hooking up other 3rd
party services such as Sumologic, Datadog, and/or NewRelic for error reports. I have always thought
that it would be a dream if all of these was in one application. And I guess my dream has finally
been heard by the gods and they have given me [Google Stackdriver](https://cloud.google.com/stackdriver/docs/)

<iframe src="https://giphy.com/embed/dUNoNFUOYV8Yw" width="480" height="270" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>
<p><a href="https://giphy.com/gifs/graph-dUNoNFUOYV8Yw">via GIPHY</a></p>

Right off the bat, when you release an application on Google App Engine, you get system reports,
which is in most cases good enough. The response monitoring, error logs are in most cases what you
get for the paid plan of the other 3rd party services. But for $8 a month, you get all that, plus
more.

Ever had a theory that the bottleneck of your app is that on nasty method that makes all the queries
really slow but have no way to prove it? Google Stackdriver Trace can do all of that for you. Here's
a [talk](https://www.youtube.com/watch?v=U6weNkNmC7s) that demonstrates the power of installing
one gem in your Rails application.

## Deploying a Rails Application
Before you can deploy a Rails application to AWS, you'll have to learn how to use cloudformation,
route53, EC2, Load Balancing, Auto Scaling, VPC's, and write a UserData. And while that gives you
a lot of flexibility, the barrier of entry is pretty steep. Because of that, I've always just
recommended using [Heroku](https://heroku.com) because if Heroku is becoming too expensive, then
you are likely in a position to hire an ops team to migrate to AWS or you're application is doing
well enough to pay for the price.

Google App Engine is almost as easy as using heroku, but having the power of AWS. All I had to do
to deploy a trial Rails application that scales automatically was to install 1
[gem](https://github.com/GoogleCloudPlatform/appengine-ruby) and write 9 lines of `YAML`. Their
tutorial to deploy a sinatra app can be done in less than 5 minutes. My jaw dropped on my first
time of deploying it.

<iframe src="https://giphy.com/embed/xT77XWum9yH7zNkFW0" width="480" height="270" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>
<p><a href="https://giphy.com/gifs/9jumpin-wow-nice-well-done-xT77XWum9yH7zNkFW0">via GIPHY</a></p>

---

While I think that AWS will still dominate the space for a few more years because all their niche
products, I'd really consider looking at the Google Cloud Platform. It may not be for your company
or your startup but at least you've seen the other side.

What do you or your company currently use to deploy your applications? Do you use Google Cloud and
not like it? I'm interested to hear more about it! Tweet me, my handle is @pauloancheta.

BTW, Google is not paying me to write all this. I wish they did but they don't. Just sharing what
I have experienced using their products.
