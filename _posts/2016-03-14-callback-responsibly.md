---
layout: post
title: "Callback Responsibly"
date: 2016-03-14 8:00
tags:
  - rails
  - callbacks
shared_description: "Avoiding callbacks makes it easier to test and to code!"
---
Recently, I have worked on a project where one of the biggest obstacle for me was to understand the app's business logic due to its callback chains. Whenever I create a model, I get an error because it tried to validate something that was supposed to be created during a callback. At that time, I found myself in what it seemed to be a rabbit hole that does not end. (Ok, I might have been exaggerating because it was probably just about 5 callback chains. MAYBE. I don't really want to count). It got a bit frustrating and I knew that there had to be a better way.

<!-- break -->

I asked one of my mentors ([Philippe](https://twitter.com/pcreux)), the most important question: __When should we use callbacks?__ And his short answer was: _"You don't"_. In my opinion, having callbacks in a Rails app can easily get out of hand. He explained that previously, the __Rails way__ was to have a _Fat Controllers_. But after that, then came the __Fat Models, Skinny Controllers__ idea where we have to basically say, _"After we create the user account, let's send him an email or notify Intercom! We should do this by adding more methods on our models that calls other classes and attach it to a callback!"_.

I believe that they soon realized that apps that uses this pattern is becoming a nightmare to work on, because the callbacks makes the models harder to test. When adding features to the models, they also have to touch the specs. Soon after that, __Service Objects__ became the new hip thing to use because it encourages extraction of the action of a model. So, the question is, how do service objects help with creating other models? Consider this:

{% highlight ruby %}
# app/models/dog.rb
class Dog < ActiveRecord::Base
  has_many :collars
  after_create :create_collars!

  def create_collars!
    collars.create!
  end
end

# app/models/collar.rb
class Collar < ActiveRecord::Base
  belongs_to :dog
  before_create :register_to_fitbit!

  def register_to_fitbit
    FitBitCollar::Create.call
  end
end
{% endhighlight %}

When writing a test for the `Dog` model, you would have to write a valid `Collar` instance with all the attributes to register it to FitBit (let's just say he is a hipster dog) before you can actually test the `Dog`! Now imagine if there were 5 callbacks that need to run when the `Dog` is created. It will most likely take a long time to run since the `Dog` is tightly coupled with other classes and all its dependencies. Also, the test would rely on other objects that shouldn't be part of the spec.  Now let's see how it looks like with a service object:

{% highlight ruby %}
# app/models/dog.rb
class Dog < ActiveRecord::Base
  has_many :collars
end

# app/models/collar.rb
class Collar < ActiveRecord::Base
  belongs_to :dog
end

# app/services/dog/create.rb
class Dog::Create
  def call(params={})
    dog = Dog.new(params)

    Dog.transaction do
      dog.save!
      collar = dog.collars.create!
      register_to_fitbit!(collar)
    end
  end

  private
  def register_to_fitbit!(collar)
    FitBitCollar::Create.call(collar: collar)
  end
end
{% endhighlight %}

When writing a test for the  `Dog` class, you actually don't have to create a valid instance of a `Collar` because all its association dependencies are extracted to a service! When testing a model, you should not care about other models, the test should focus on the `Dog` model and not care about the dependencies that the dog may have. This makes testing the class so much easier and makes the testing suite so much faster which is a huge win!

So now, the question is, "Are there good uses for callbacks?". The answer is a big YES! Callbacks are really useful in tasks affecting the model layer (not the business logic) like updating a column of the parent class.

{% highlight ruby %}
# app/models/dog.rb
class Dog < ActiveRecord::Base
  has_many :collars
end

# app/models/collar.rb
class Collar < ActiveRecord::Base
  belongs_to :dog
  after_save :make_main_swag!

  def make_main_swag!
    dog.update!(main_swag_id: id)
  end
end
{% endhighlight %}

With this code, everytime we update the collar of a dog, it becomes the collar that our dog is going to use.

Without callbacks, testing is generally easier and can give an overall better development experience. Use callbacks responsibly and please don't create new records with callbacks, instead step back and think if it is really necessary to have such a dependency between records. If they really have to be dependent on each other, use a service object to extract the action and use a transaction!
