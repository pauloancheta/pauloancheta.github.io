---
layout: post
title: "Robust JSON API for Mobile Apps Powered by Rails"
date: 2016-04-22
categories:
  - base64-images
  - jsonapi-resources
  - rspec_api_documentation
description: "Uploading Base 64 images using jsonapi-resources with good and up to date documentation"
---

Let’s say we have a [Rails](http://rubyonrails.org) app that uploads images using `carrierwave`. We want to extend this functionality to let a mobile app upload images as well. The only constants we know are that the photos should be sent to our Rails app through a RESTful JSON API, and that the images are strings encoded in base64.

<iframe src="//giphy.com/embed/TFMoOxjnAAMbm" width="480" height="269" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="http://giphy.com/gifs/television-lol-TFMoOxjnAAMbm">via GIPHY</a></p>

<!-- break -->

# Here are the tools that we are going to use
- [carrierwave-base64](https://github.com/lebedev-yury/carrierwave-base64) - Upload files encoded as base64 to carrierwave.
- [jsonapi-resources](https://github.com/cerebris/jsonapi-resources) - provides a framework for developing a server that complies with the JSON API specification.

_For The Stretch Goal_

- [rspec_api_documentation](https://github.com/zipmark/rspec_api_documentation) - Generate pretty API docs for your Rails API
- [apitome](https://github.com/modeset/apitome) - Rails viewer for the documentation

# Add Models to DB and mounting the uploader
_Implementing carrierwave-base64_

We need a place to store the images. We could use a generator to create a `Post` table that has an `image` column which stores strings.
{% highlight ruby %}
>$ rails g model post image:string
>$ rake db:migrate
{% endhighlight %}

With our generated model, we attach a base64 image uploader which will allow us to attach an object in the fields of our database. The code still looks like an ordinary `carrierwave` implementation -- but with a really small difference. Instead of having `mount_uploader` in the model, we would add `mount_base64_uploader` instead.

{% highlight ruby %}
# app/models/post.rb
class Post < ActiveRecord::Base
  mount_base64_uploader :image, ImageUploader
end

# app/uploaders/image_uploader.rb
class ImageUploader < CarrierWave::Uploader::Base; end
{% endhighlight %}

{% highlight ruby %}
# rails console
p = Post.new
base64_image = Base64.encode64(File.read(awesome_picture.jpg))
p.image = "data:image/jpg;base64,#{base64_image}"
p.save!
{% endhighlight %}

Now that can save a base64 image, we now have to create an API endpoint that our mobile app can call so they can post images.

# Creating the JSON API Endpoint
__implementing jsonapi-resources__

Although there is a lot more to explore in `jsonapi-resources`, I will only touch on just a few of its really cool features. I believe this gem deserves its own blog post on how much benefit it provides with just a few lines of code.

Now let's create a `jsonapi-resources` controller and resource with generators that the gem provides.
{% highlight ruby %}
>$ rails generate jsonapi:resource api/post
# => app/resources/api/post_resource.rb
class Api::PostResource < JSONAPI::Resource
  attribute :image
end

# app/controllers/api/application_controller.rb
class Api::ApplicationController < JSONAPI::ResourceController
  protect_from_forgery with: :null_session
end

# app/controllers/api/posts_controller.rb
class Api::PostsController < Api::ApplicationController; end

# config/router.rb
namespace "api" do
  jsonapi_resources :post, only: [:create]
end
{% endhighlight %}

Although the `ApplicationController` that we have written inherits from the `jsonapi-resources` controller, this can also be a normal controller that includes a `ActsAsResourceController`.

In the routes, we are using the `jsonapi_resources` method. This gives us a lot of useful endpoints. For the sake of this example, let's just focus on a posting endpoint and add `only: [:create]`. Thus giving:
{% highlight javascript %}
api_posts POST   /api/posts(.:format)           api/posts#create
{% endhighlight %}

This is actually all we need to post a base64 image through an API. From here we can use Postman:
{% highlight javascript %}
curl -X POST -H "Content-Type: application/vnd.api+json" -H "Cache-Control: no-cache" -H "Postman-Token: 233cdeb0-ba65-7bd5-c550-8e8b79e181bb" -d '{
  "data": {
      "type": "posts",
      "attributes": { "image": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAMgAAADICAMAAACahl6sAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAA2hpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtcE1NOk9yaWdpbmFsRG9jdW1lbnRJRD0ieG1wLmRpZDpCQTg1MENEQjEyMjA2ODExQjI2OUUwNTczQjFGQjMxMyIgeG1wTU06RG9jdW1lbnRJRD0ieG1wLmRpZDoyQjU5RTEyQkM3NjAxMUU0QTQ1NUJBOTY0QzkzRDVCMiIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDoyQjU5RTEyQUM3NjAxMUU0QTQ1NUJBOTY0QzkzRDVCMiIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ1M2IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6QkI4NTBDREIxMjIwNjgxMUIyNjlFMDU3M0IxRkIzMTMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6QkE4NTBDREIxMjIwNjgxMUIyNjlFMDU3M0IxRkIzMTMiLz4gPC9yZGY6RGVzY3Jp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" }
  }
} ' "http://localhost:3000/api/posts"
{% endhighlight %}

---

# Stretch Goal: Testing & Documentation
_rspec api documentation_

It is important to test and document API implementations. With `rspec_api_documentation`, we can do both at the same time. In my opinion, the best part of using this gem is that it does not generate the documentation for a failed example. It runs all the acceptance test, and if it passes, it generates the documentation. Once the documentation is re-generated, all the documentation is removed and generates a new one. Also, example documentation can be skipped with the `document: false` option.

First we need to tell `rspec_api_documentation` that we are going to be formatting the body to a `JSON` response by adding this helper:

{% highlight ruby %}
# spec/rails_helper.rb
# Values listed are the default values
RspecApiDocumentation.configure do |config|
  # Change how the post body is formatted by default, you can still override by `raw_post`
  # Can be :json, :xml, or a proc that will be passed the params
  config.request_body_formatter = :json
  config.format = :json
end
{% endhighlight %}

Now that is all set up, we can start writing our test.

To set up our test, we would first have to include `rspec_api_documentation` dsl. This gives us wrappers to have headers to our requests and setting HTTP verbs as context. We also use `resource` instead of `describe` to define what we are testing.

{% highlight ruby %}
# spec/acceptance/post_spec.rb
require "rails_helper"
require "rspec_api_documentation/dsl"

resource "Posts" do
  let!(:valid_base64_image){ Base64.encode64(File.read(awesome_picture.jpg)) }
  let!(:request_attributes){
    {data: {type: "posts", attributes: {image: "data:image/png;base64,#{valid_base64_image}"}}
  }

  header "Accept", "application/vnd.api+json"
  header "Content-Type", "application/vnd.api+json"
end
{% endhighlight %}

Below I have added a method that will be passed in a request method in my "example" ("example" in an acceptance test is analogous to an `it` block). A header method takes in 2 arguments: the header field name as a string and the header value.

{% highlight ruby %}
resource "Posts" do
  # ... lines ommitted

  post "/api/posts" do
    example "Post a photo" do
      do_request(request_attributes)
      expect(status).to eq 201

      images = JSON.parse(response_body)
                   .fetch("data")
                   .fetch("attributes")
                   .fetch("image")
      expect(images["image"]["url"]).to be_present
    end
  end
end
{% endhighlight %}

Now, let's examine what goes in the test. In `rspec` we normally use `describe` or `context`. In an `rspec_api_documentation` test, we use the http verb, followed by the path that we want to test. It also takes in a block that contains a `do_request` method. This method can take in an argument. In a `GET` request, it does not need an argument, but for our case, the `POST` request takes in a hash as an argument.

Run the test and generate the docs with:
{% highlight ruby %}
>$ rake docs:generate

# Or jus run the test without generating the documentation
>$ rspec spec/acceptance
{% endhighlight %}

The docs are available at `http://localhost:3000/docs/api` and with the help of `apitome` this would look really cool! In essence, `apitome` is a wrapper for `rspec_api_documentation` to enhance the generated documentation.

---

Creating an API endpoint is never complete without a good proper documentation. With this API bootstrap combo for Rails, it makes mobile image upload feature easier. See how awesome this combo is with my toy app! Follow this [link](https://blog-jsonapi.herokuapp.com/) to go to the website, and this [link](https://blog-jsonapi.herokuapp.com/docs/api) to go to the api documentation

