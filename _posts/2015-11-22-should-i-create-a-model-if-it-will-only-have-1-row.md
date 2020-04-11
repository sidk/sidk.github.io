---
id: 116
title: Should I create a model if it will only have 1 row?
date: 2015-11-22T16:28:12-05:00
author: Sid
layout: post
guid: http://ducktypelabs.com/?p=116
permalink: /should-i-create-a-model-if-it-will-only-have-1-row/
cta_content_placement:
  - below
categories:
  - ActiveRecord
---
A post on Reddit says:

> I&#8217;m not sure how best to do this. I would just set up a model, but that seems wrong to have a model with just 1 row in it (it&#8217;s a single charity). Is there a better way to do this? STI? something else?

It can seem like overkill to create a database table and a corresponding ActiveRecord model if you know for a fact that your table will only have one row. This is a pretty common situation, and there are a couple of ways to deal with this.

Note: [Pat Maddox&#8217;s](http://www.patmaddox.com/) [amazing comment on Reddit](https://www.reddit.com/r/rails/comments/3jxo4d/help_should_i_create_a_model_if_it_will_only_have/), which addresses this question, was the primary inspiration for this post.

First, if **non-developers** need the ability to update information that this &#8220;model&#8221; would contain, then yes, you should create an ActiveRecord model. This will of course give you all the benefits of ActiveRecord as well, like being able to use `has_many`, `belongs_to` and everything else.

If **only developers** are going to be concerned with and using this model, **you don&#8217;t have to tie it to the database and can implement it as a &#8220;Plain Old Ruby Object&#8221; aka a &#8220;PORO&#8221;**.

The advantage with PORO&#8217;s is that they are cheap to instantiate because you won&#8217;t be making a call to the database when you do so. Setting up association-like behavior can be as simple as defining either instance or class methods on the PORO. As Pat advises in his comments however, you should always ask yourself what benefit you&#8217;d get from implementing association-like behavior in your PORO to ensure that you&#8217;re doing it for the right reasons, and to keep your code easy to understand.

So, going with the example in the Reddit post above, where we have a `Charity` model (this is the model that will only have one &#8220;row&#8221; and the one we will be writing a PORO for) which needs to define a `has_many` type association with the `Donation` model, you&#8217;d have something that looks like:

    class Charity #this is our PORO
      def name
        'The Human Fund'
      end
    
      def donations
        Donation.all
      end
    end
    
    class Donation < ActiveRecord::Base #this corresponds to a table in the DB
      validates :amount, presence: true
    
      def charity
        Charity.new
      end
    end
    

As you can see, we&#8217;re manually defining the methods (`#donations` and `#charity`) that we&#8217;d get automatically if we used `has_many`. This allows us to get around the fact that a `Charity` object does not correspond to a table in the database, and lets other parts of our code use `Charity` objects as they would any other ActiveRecord object.

For more association type goodness on POROs, also check out the [activemodel-associations gem](https://github.com/joker1007/activemodel-associations).

So if you decide to go with a PORO, you next question is probably: **Where should this file reside?**

There are a couple of common options &#8211; in the models directory with the rest of the models in /app/models, or in the /lib directory. Pat Maddox recommends the models directory, and I concur, because if you&#8217;re going to treat it like a model in other parts of your code, it&#8217;s natural that other developers (or you in the future) will look for the class in the models directory. That being said, this is just a convention, and either way works.
