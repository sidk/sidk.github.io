---
id: 354
title: When is indexing in a model appropriate?
date: 2016-05-25T14:26:01-04:00
author: Sid
layout: revision
guid: http://ducktypelabs.com/185-revision-v1/
permalink: /185-revision-v1/
---
A user on Reddit says:

> I&#8217;m not a DB ninja like a lot of developers as I started out more front end and moved towards full stack. I understand the point of indexing is to speed up SQL queries on large data sets, but I feel like I&#8217;m just not sure when to use them and when not to.
> 
> What is the current best practice with rails?

Uncertainty often stems from a lack of understanding, and indexes in your Rails app can, if you&#8217;re not familiar with them, be akin to those murky backwaters you rarely venture into. But it needn&#8217;t be. Indexing done right can make a huge difference to the performance and speed of your app, which more often than not, will translate into increased sales, conversions or whatever it is you measure to determine if your app&#8217;s ultimate purpose is being fulfilled.

## What is an index?

An index is a database construct that makes it easier for the database to look up a given piece of information. An oft cited example, which brings the point home, is that of a phone book index. Imagine you wanted to look up one Rick Sanchez&#8217;s phone number &#8211; what would be faster? Looking through each page of the book one by one, or flipping back to the index which conveniently lists in alphabetical order the page number Rick&#8217;s phone number might be found in? Of course, the index would be way faster. And the positive performance effects multiply the more times you have to look up phone numbers.

Now, when someone moves in or out of Rick&#8217;s locality (which the phone book covers), or someone&#8217;s name or phone number changes, the index at the back will also have to be updated. This _might_ end up costing you performance if you&#8217;re not careful.

## Why use an index?

If we represented the phone book example in database terms, we&#8217;d probably have a `PhoneBook` table, with a `name` and `phone_number` column. Adding an index to the table on the `name` column, makes it easier and faster for the database to look up a phone number for a given name (say when you do something like `PhoneBook.where(name: 'Morty Smith')` ), especially if there are a large number of rows to look through.

**So the main reason to use an index on a given column is increased speed, which will more often than not, translate into more sales, conversions or whatever it is you measure to determine if your app&#8217;s ultimate purpose is being fulfilled.**

Note that although we&#8217;re just talking about indexing on one column, it is actually possible to index on multiple columns too. I&#8217;ll list out resources at the end of this article which will go more in depth into topics like this.

## When to use an index

If you&#8217;re with me so far, it&#8217;s clear to you that an index on a column makes it faster for the database to look up a row or a set of rows which satisfy a given condition on that column. With that, we can come up with a rule of thumb:

**You should add an index on a column if most (or many) of the queries you write for the table involve that column.**

In a typical Rails application, there&#8217;s one type of column that is pretty much guaranteed to be involved in most queries for tables that it is in, and that is the [foreign key column](/all-about-foreign-keys).

Let&#8217;s consider an example.

Say you have a `User` and a `Post` model, and the `User` `has_many :posts`. This of course means that the `:posts` table has a foreign key column `user_id`, and that whenever we do something like `@user.posts` or `@post.user`, we are asking the database to look up a given `user_id` for one or many `Post` records. Without an index, the database would have to look up each row in the `posts` table, one by one, to see if their `user_id` matched what was given. If there are many `Post` records (in other words, if `Post.count` returns a huge number), then this would take a lot of time. So, we would add an index to the `posts` table on `user_id`.

Right now, you&#8217;ll have to take my word for it that this is an improvement. But I&#8217;ll cover how you can verify this for yourself very soon.

So, **add indexes by default to your foreign key columns. This is considered a standard Rails practice.**

&#8220;Non-foreign key&#8221; columns can also be good candidates for adding indexes on. If they&#8217;re referred to a lot in your table queries, and you have a sizable number of rows in your table, then it would probably benefit you to add an index on them. Common examples I&#8217;ve come across in my experience are the `date_of_birth`, `email`, `name` columns in a `User` table. Remember to apply the rule of thumb discussed above if you&#8217;re unsure.

## How to Index

You know what an index is and when to use it. Here&#8217;s where we cover how you can begin to actually use them in your Rails app.

### When you&#8217;re associating two models with no prior connection

If you have two models which you want to associate, you know that you have to create a foreign key column on one of them. It turns out that in Rails 4 and beyond, you can create an index at the same time (which you would want to do in most cases), with the [`add_reference`](http://apidock.com/rails/ActiveRecord/ConnectionAdapters/SchemaStatements/add_reference) method.

Say you&#8217;ve created a `User` and `Post` model, and you want to set up a `has_many` relationship between them in your DB, you could do this in your console:

    rails g migration AddUserToPosts user:references
    

which will generate a migration which uses the &#8216;add_reference\` method like so:

    add_reference :posts, :user, index: true
    

The `add_reference` method will not only create a `user_id` column on your `Post` table, but also add an index on `user_id`. Nifty. You can also pass in `foreign_key: true` to this method to add in a [database constraint](/all-about-foreign-keys)

### Adding an index to an existing association

If for some reason, you don&#8217;t have access to the `add_reference` method, or you&#8217;ve already created a `user_id` column without adding an index to it, you can use the [`add_index`](http://apidock.com/rails/ActiveRecord/ConnectionAdapters/SchemaStatements/add_index) method in your migration like so:

    add_index :posts, :user_id
    

You can use the `add_index` method to add an index on any column in the table.

## How to check if your index made a difference

So far, you&#8217;ve taken my word that indexing is a good thing which speeds up your app in ways that are relevant to business goals. But you don&#8217;t have to, because **you can use `ActiveRecord's` [`#explain`](http://guides.rubyonrails.org/active_record_querying.html#running-explain) method to verify if adding an index made a difference to your query time or not**. This will also help you determine the situations where indexing _doesn&#8217;t_ give you much of a performance benefit.

The `#explain` method in `ActiveRecord` maps to the `EXPLAIN` command in `SQL`, and one of the things this command does is tell you how long the database thinks the query in question will take. The specifics of how the database formats this &#8220;explanation&#8221; and the units it measures &#8220;execution time&#8221; in depend on the database you&#8217;re using. Two common databases in the Rails world are [Postgres](http://www.postgresql.org/docs/current/static/using-explain.html) and [MySQL](http://dev.mysql.com/doc/refman/5.6/en/explain-output.html), so I&#8217;ll give you the basics on those.

To see the output of the `#explain` method, you simply send the `#explain` message to the query you&#8217;re writing, like so:

    Post.where(user_id: 23).explain
    

If you&#8217;re on a **Postgres** database, and you&#8217;ve run explain on a query, you&#8217;ll want to look at the **cost** numbers. The higher this number is, the longer the database thinks your query is going to take. You&#8217;ll typically see a range being output for cost.

If you&#8217;re on a **MySQL** database, you&#8217;ll want to look at the **rows** number, which tells you how many rows the database thinks its going to have to go through to get you a result. The higher this number, the longer the database thinks your query is going to take.

**In short, by using &#8216;#explain&#8217; before and after you&#8217;ve added an index, you can see for yourself how much of a difference adding the index has made.**

[This article](https://tomafro.net/2009/08/using-indexes-in-rails-index-your-associations), and [this one](https://tomafro.net/2009/08/using-indexes-in-rails-choosing-additional-indexes), written by [Tom Ward](https://tomafro.net/), a Rails contributor and developer at [Basecamp](https://basecamp.com/), are fantastic reads which go deeper into indexing and show you how you can make sense of the `explain` method&#8217;s output (he uses MySQL in his examples).

[Tom also has this handy script](https://tomafro.net/2009/09/quickly-list-missing-foreign-key-indexes), which you can use to list any foreign keys you might have missed indexing.

## Advanced Topics

Indexing is a vast topic, and if you want to dig deeper, here are some starting points:

### [A free book on indexing](http://use-the-index-luke.com/)

This book covers many things I didn&#8217;t cover here. Definitely worth a read.

### When not to use an index

  1. [Check this out](http://searchsqlserver.techtarget.com/feature/When-not-to-use-indexes), and 
  2. [this](http://dba.stackexchange.com/questions/56/how-to-determine-if-an-index-is-required-or-necessary).

### Optimizing Indexes

  1. [Tips on optimizing indexes](http://www.sql-server-performance.com/2007/optimizing-indexes-general)

### Terms to Google

  1. Partial Indexes
  2. Redundant Indexes

As always, hit me up in the comments section if you&#8217;re stuck on anything in particular or if there&#8217;s a topic you want me to write more about. Cheers, and until next time!