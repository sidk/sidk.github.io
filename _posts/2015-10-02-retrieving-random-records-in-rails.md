---
id: 60
title: Retrieving Random Records in Rails
date: 2015-10-02T14:31:50-04:00
author: Sid
layout: post
guid: http://ducktypelabs.com/?p=60
permalink: /retrieving-random-records-in-rails/
cta_content_placement:
  - below
categories:
  - ActiveRecord
---
### What is the best way to retrieve one or more random records in Rails?

Given that there are multiple ways to retrieve random records in Rails and Ruby ([take a look at this SO post](http://stackoverflow.com/questions/5297396/quick-random-row-selection-in-postgres)), it can be confusing and frustrating to decide which course of action to take.

> &#8220;Is there a technique to retrieve random records that is correct/general/Rails way/cross-database compatible? &#8220;

In this post, I&#8217;ll cover a couple of simple approaches that you can take to figure out what the best way might be for you.

The first priority that you likely have to keep in mind is speed. Most applications have a threshold for what is considered slow &#8211; in web apps for example, anything that takes longer than a few hundred milliseconds to load is considered slow. So you don&#8217;t want to be choosing a method that will have your users waiting around forever for your app to do something. That being said, there is also no point in sacrificing code readability, ease of re-use and just time in general, to get faster than you have to.

The second thing you have to consider is the size (number of rows) of the database table you are looking to retrieve a random record from. In general, the bigger your table, the more knowledgable you&#8217;ll have to be about your database platform (typically Postgres or MySQL) to get to your desired speed.

Take this common solution to retrieve a random record in Rails:  

    random_user = User.all.sample
    

To use the code above requires no internal knowledge of the database you&#8217;re running on. The #all method is a well-known ActiveRecord method and #sample is a Ruby method on Array that retrieves a random value from a given array.

The flip side of this approach is that if your database has a sizable number of rows, it will take a long time to execute. Try it out on your Rails console and see! The reason it does so is because `User.all` retrieves _all_ the users from the database, and then builds ActiveRecord objects from them. So if you have let&#8217;s say 10,000 users, building 10,000 ActiveRecord objects takes a while.  If you&#8217;re reasonably certain that the table in question will not grow much in size over the lifetime of your app, and the speed at which the above code executes is agreeable to you, then by all means go for it.

Now, if your table is HUGE (has a million+ rows for example), the above method is not going to work for you. Consider using the ActiveRecord #offset and the Ruby rand methods. The #offset method in ActiveRecord specifies how many rows to skip before returning the rest of the rows. The rand method, as expected, generates a random number less than a maximum value &#8211; so when you say `rand(10)`, it will generate a random number less than 10.

So something like this:

    count = User.count
    random_offset = rand(count)
    random_user = User.offset(random_offset).first`
    

&#8230;works better (faster) for big tables, because you&#8217;re not making the database scan through every single row in the table.

Resources:

  * [Tips on Retrieving Random Records](http://hashrocket.com/blog/posts/rails-quick-tips-random-records) from the folks at HashRocket. Make sure you check out the comments section too
  * [Stackoverflow question on retrieving Random Records](http://stackoverflow.com/questions/5297396/quick-random-row-selection-in-postgres). This is Postgres specific, but has some good ideas that apply to MySQL as well