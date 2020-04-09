---
id: 136
title: Using ActiveRecord scopes with associations
date: 2015-11-27T14:06:53-05:00
author: Sid
layout: revision
guid: http://ducktypelabs.com/128-revision-v1/
permalink: /128-revision-v1/
---
When you have a `belongs_to` or a `has_one/has_many (with or without :through)` association defined on your model, you will in many situations benefit from defining scopes in your model that reference your association. Defining a custom scope which references attributes in your association table(s) has the potential to DRY up your code and make code in other parts of your application easier to understand.

However, implementing such scopes can cause much confusion and frustration; especially when you see hard to interpret `SQL-y` errors being returned, and have no idea how to go about fixing them.

### Defining scopes directly on the association

`ActiveRecord's` `has_many` and `belongs_to` methods give you the ability to define scopes directly on the association. This way, you can customize the `SQL` that is generated when you access the association.

Say you have a `User` model which `has_many` `posts`, and when you do `@user.posts`, you only want to return the posts which were created in the last year. Your `User` model would look like:

    class User < ActiveRecord::Base
      has_many :posts, -> { where('created_at > ?', Time.current - 1.year) }
    end
    

Because you&#8217;ve defined the scope this way, when you do `@user.posts`, the generated query will include the `where` condition above so that any posts returned will have been created within the past year.

You can also pass in an argument to this scope, which allows you to further customize the generated query. Say instead of defaulting to 1 year you wanted to pass in the timeframe, you could do something like:

    class User < ActiveRecord::Base
      has_many :posts, ->(timeframe) { where('created_at > ?', timeframe }
    end
    

One gotcha to keep in mind, especially when using columns like `created_at` which are common to most tables, is that you might have to specify the table name explicitly in your scope if your DB complains. So the above would look like:

    class User < ActiveRecord::Base
      has_many :posts, ->(timeframe) { where('posts.created_at > ?', timeframe) }
    end
    

### Referencing attributes of the association table in your scope

You might also sometimes benefit from defining scopes in the parent model which reference attributes of your association. Adding to the example above, let&#8217;s say you wanted to define a scope on the `User` model which returns all users which have pending posts (posts which haven&#8217;t been &#8220;approved&#8221; yet).

    class User < ActiveRecord::Base
      has_many :posts
      scope :with_pending_posts, -> { joins(:posts).where('posts.pending = true') }
    end
    

Couple of things to note from the scope definition above:

  1. You have to use the `joins` method in your scope so that your DB knows which table you&#8217;re referring to.
  2. To reference association attributes in the `where` method, you have to use the table name of the association followed by the column name (`posts.pending` in this case) to prevent your DB from complaining, and to have it return the expected result.

A corollary from the points above is that you can technically reference attributes of any table in your scope, even if they are not explicitly defined as an association in your model. You&#8217;d of course have to set up your `joins` correctly &#8211; refer to [the ActiveRecord documentation](http://apidock.com/rails/ActiveRecord/QueryMethods/joins) for more info, and if you&#8217;re still stuck, ping me in the comments below!

### Troubleshooting error messages

When the scope you&#8217;re building is complex, you&#8217;ll probably run into errors, especially `SQL` errors. It will benefit you greatly to learn how to read these errors and understand what your DB is complaining about. Typical complaints include tables which are not defined and columns which don&#8217;t exist because the query is looking for them in the wrong table.

Remember that a custom scope is the same as defining a class method. So if you&#8217;re stuck, ask yourself how you would implement the scope as a class method.

Keep in mind that if you want your scope to do calculations based on association attributes, you might not be able to use `Ruby`, and will have to brush up on your `SQL` (more importantly, `SQL` that works with your DB of choice) to accomplish what you need. For example check out the scope below, where for an `Availability` model, we&#8217;re returning records whose `start_time` is greater than an associated `Venue's` `notice_time`, which in this case happens to be stored as an integer. As you&#8217;ll notice, `Now()` and `1 hour::interval` are `Postgres` specific SQL.

    class Availablity < ActiveRecord::Base
      has_one :venue
      scope :after_notice_time, -> { joins(:venue).where("start_time >= now() + venues.notice_time * '1 hour'::interval") }
    end
    

Check out this [Reddit thread](https://www.reddit.com/r/ruby/comments/3s7l1y/referencing_self_and_through_association_in_scopes/) for more details on the above example.

As always, let me know in the comments how I can help you, and if you liked this article, subscribe to my email list below for updates on whenever I post a new article. Cheers! [mc4wp_form]