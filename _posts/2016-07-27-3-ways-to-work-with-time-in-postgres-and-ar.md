---
id: 410
title: '3 ways to work with time in Postgres (&#038; ActiveRecord)'
date: 2016-07-27T16:11:04-04:00
author: Sid
layout: post
guid: http://ducktypelabs.com/?p=410
permalink: /3-ways-to-work-with-time-in-postgres-and-ar/
cta_content_placement:
  - below
classic-editor-remember:
  - classic-editor
categories:
  - ActiveRecord
---
Let&#8217;s say you&#8217;re developing a project management app with users and projects and the product manager asks you:

>   1. How much time does it take a given user to create their first project?
>   2. Give me a list of users who took longer than 1 month to create their first project.
>   3. What is the average amount of time that it takes for a user to create their first project?
>   4. What is the average amount of time per user between project creation?

You could approach solving the problems above purely in Ruby and `ActiveRecord`. However the data we want to work with resides in a database. The database is smart, capable & fast; with SQL, we can make the database retrieve arbitrary rows and perform complex calculations. It therefore stands to reason that making the database &#8220;do the work&#8221; as much as possible before sending information to Ruby-land will net us performance gains.

In this article, we&#8217;re going to explore how we can solve the above problems in Rails using `SQL` and a bit of `ActiveRecord`. The SQL used in this article applies only to `Postgres`(version 9.4.1), though the ideas in general can be translated to other database management systems.

## First, a brief primer on `SELECT`

A `SELECT` statement retrieves zero or more rows from one or more database tables or database views.

So something like:

    SELECT * FROM users
    

will retrieve all available rows from the `users` table.

You can also pass in &#8220;aggregator&#8221; functions like MIN(), COUNT(), MEAN() etc to SELECT. When these functions are passed in to SELECT, the return value is a single piece of data (an integer, string, timestamp etc).

So for example this:

    SELECT MIN(created_at) FROM users
    

will return the earliest `created_at` (which is a timestamp) in the users table.

`SELECT` functions can also be nested within themselves, as we&#8217;ll see soon enough.

In `ActiveRecord` the [`select` method](http://api.rubyonrails.org/classes/ActiveRecord/QueryMethods.html#method-i-select) can be used to achieve the same result. Also, calls to methods such as `.all`, `.where`, `.find` etc translate to `SELECT` functions.

**I&#8217;ve prepared a handy 3-page PDF cheatsheet containing the salient points of this article. If you&#8217;d prefer downloading and printing it out, [click here to get it!](/wp-content/uploads/2016/07/3_ways_to_work_time_in_postgres.pdf)**

For the sake of this article, assume our models look like this:

    class User < ActiveRecord::Base
      has_many :projects
    end
    
    class Project < ActiveRecord::Base
      belongs_to :user
    end
    

## Arithmetic operators (`-`, `+`, `*` and `/`)

The first problem we have to tackle is figuring out how much time it takes a user to create their first project. This time equates to the difference between the user&#8217;s earliest `Project#created_at` and `User#created_at`.

In Postgres, we can use the arithmetic difference operator `-` to calculate the difference between two timestamps or dates. The data type matters &#8211; if we calculate the difference between two timestamps, the return value is an &#8220;interval&#8221;, and if we calculate the difference between two dates, the return value is an integer representing the number of days between the two dates.

To make it easy to display the data, we&#8217;ll plan to use the `select` method with something like:

    users = User.select('users.email, (projects.created_at - users.created_at) as time_to_first_project')
    

This will return an `ActiveRecord::Relation` which we can then iterate over and call `#time_to_first_project` on:

    <% users.each do |user| %>
      <%= user.email %>
      <%= user.time_to_first_project %>
    <% end %>
    

However, the above `select` won&#8217;t work for us for two reasons. First, ActiveRecord will complain that there is no `FROM` clause for the `projects` table. We can solve this with a call to `joins`:

    User.joins(:projects).select('users.email, (projects.created_at - users.created_at)....')
    

Second, the above statement compares the creation times of _all_ of a given users&#8217; projects to `User#created_at`. We only care about the earliest project, so we can narrow this down with a call to `where`:

    User.joins(:projects)
        .where('projects.created_at = (SELECT MIN(projects.created_at) FROM projects WHERE projects.user_id = users.id)')
        .select("users.email, (projects.created_at - users.created_at) as time_to_first_project")
    

Because a user&#8217;s earliest project&#8217;s `created_at` varies with the user, we retrieve this timestamp by passing in the aggregator function `MIN()` to a nested `SELECT` function (nested in this case within the first `select`). This nested `SELECT` function is also known as a &#8220;sub-query&#8221;.

This will return a collection of `ActiveRecord` objects with the attributes `email` and `time_to_first_project`. Because we&#8217;re subtracting two timestamps, `time_to_first_project` will be an &#8220;interval&#8221;.

[Intervals in Postgres](https://www.postgresql.org/docs/9.4/static/datatype-datetime.html) are the largest datatype available for storing time and consequently contain a lot of detail. If you inspect `time_to_first_project` for a few records, you&#8217;ll notice that they look something like: `"00:18:43.082321"` or `"9 days 12:48:48.220725"`, which means you might have to do a bit of parsing and/or decorating before you present the information to the user.

Postgres also makes available the `AGE(..)` function, to which you can pass in 2 timestamps and get an interval. If you pass in one timestamp to `AGE()`, you&#8217;ll get back the difference between the current date (at midnight) and the passed-in timestamp.

If you have two timestamps and you want find the number of days between them, then you can use the `CAST` function to take advantage of the fact that when you subtract two dates you get the days between them. We&#8217;ll come back to `CAST` in a little bit.

## Comparison and Filtering

Next, we want a list of users who took longer than 1 month to create a project. Ideally, we&#8217;d want to be able to get a list of users who took longer than any arbitrary period of time to create a project. This operation boils down to a &#8216;greater-than&#8217; comparison between `time_to_first_project` and the time period we pass in (1 month for example). Postgres supports the comparison of dates and times to each other. In this case, since `time_to_first_project` is an interval, we have to make sure that we compare it to an interval in our call to `where`. This means that if we do:

    users.where("(projects.created_at - users.created_at) > 30")
    

Postgres will complain that the operator to compare an interval and integer doesn&#8217;t exist. To get around this, we have to ensure that the right hand side of the comparison is an interval. We do this with the [`INTERVAL` type keyword](https://www.postgresql.org/docs/9.4/static/sql-syntax-lexical.html#SQL-SYNTAX-CONSTANTS-GENERIC). We&#8217;d use it in this way:

    User.joins(:projects)
        .where('projects.created_at = (SELECT MIN(projects.created_at) FROM projects WHERE projects.user_id = users.id)')
        .where("(projects.created_at - users.created_at) > INTERVAL '1 month'")
        .select(...)
    

As you can see, we can pass in a human readable string like &#8220;1 month&#8221; to `INTERVAL`, which is nice.

## Aggregations

Next on the list, we want to calculate the average amount of time it takes to create a project. Given what know so far, this won&#8217;t require much explanation. Our query will look like this:

    User.joins(:projects)
        .where('projects.created_at = (SELECT MIN(projects.created_at) FROM projects WHERE projects.user_id = users.id)')
        .select("AVG(projects.created_at - users.created_at) as average_time_to_first_project")
    

We&#8217;re using the `AVG()` function and our query will return one `User` object on which we can call `average_time_to_first_project`.

`ActiveRecord` also has available the `average` function, which we can call like this:

    User.joins(...)
        .where(...)
        .average('projects.created_at - users.created_at)
    

This query&#8217;s return value will be a `BigDecimal`, and because of this we might lose some information. For example, if the true average is `INTERVAL "1 day 23:00:01.2234"`, the `BigDecimal` value will be `1`.

Finally, what if we want to calculate the average time between project creation for each user?

The average time between projects for a user is the average of the difference between consecutive `Project#created_at` values. So for example, if there are three projects, the average time between projects is:

    "((project3#created_at - project2#created_at) + (project2#created_at - project1#created_at))/2"
    

This is equal to `(project3#created_at - project1#created_at) / 2`. For `N` projects, the average time between projects is:

    "(projectN#created_at - project1#created_at) / (N - 1)"
    

In our example, &#8220;projectN&#8221; corresponds to the user&#8217;s latest project and therefore its creation date is equal to `MAX(projects.created_at)`. Similarly, the user&#8217;s earlier project creation date corresponds to `MIN(projects.created_at)`. The number of projects is calculated with `COUNT(projects)`.

For our output, let&#8217;s say we want the average time between projects to be reported in number of days. Since subtracting two dates in Postgres returns an integer number of days, we can obtain the average time between projects in days by first casting `MAX(projects.created_at)` and `MIN(projects.created_at)` to dates, subtracting them and then dividing by `COUNT(projects) - 1`. In Postgres, we can cast our timestamps with the `CAST` function. For example:

    CAST(MIN(projects.created_at) as date)
    

will convert the created_at timestamp to a date.

Finally, we want this calculation to be performed once for every user. For this, we have to use the `group` method, which corresponds to the `GROUP BY` clause. We need this clause because if we didn&#8217;t have it, the calculation will be performed only once for the entire set of projects, rather than for every user.

Our query will look like this:

    User.joins(:projects)
        .group('users.email')
        .having('COUNT(projects) > 1')
        .select("(CAST(MAX(projects.created_at) as date) - CAST(MIN(projects.created_at) as date))/(COUNT(projects) - 1) as avg_time_between_projects, users.email as email")
    

You&#8217;ll also notice that I&#8217;ve included a call to `.having('COUNT(projects) > 1')`. This ensures that only users who have more than one project are considered.

## Where should we put this stuff in our codebase?

### In the database

It can be beneficial to have this SQL reside in the DB instead of in your application layer. You can make this happen with views. A &#8220;view&#8221; is a stored query which we can query just as we would a table. As far as ActiveRecord is concerned, there is no difference between a view and a table.

The idea is that by creating a view with the user&#8217;s email and the fields we&#8217;re interested in, like average time between projects and time taken to create the first project, our application code will be cleaner because we can now say something like `UserReport.where('average_time_between_projects > ?', 30.days)` instead of &#8220;all the SQLs&#8221;.

To create a view, first create a migration like so:

    def change
      sql = <<-SQL
        CREATE VIEW user_reports AS
        SELECT (CAST(MAX(packages.created_at) as date) - CAST(MIN(packages.created_at) as date))/(COUNT(packages)-1) as time_between_projects, COUNT(packages) as count, users.email as email FROM \"users\" INNER JOIN \"packages\" ON \"packages\".\"user_id\" = \"users\".\"id\" WHERE \"users\".\"role\" = 'client' GROUP BY users.email HAVING COUNT(packages) > 1    
      SQL
      execute(sql)
    end
    

The SQL I&#8217;ve used was generated by calling the `to_sql` method on the query we ran previously. Run the migration and then create a `UserReport` model class which inherits from `ActiveRecord::Base`. You should now be able do things like `UserReport.average('time_between_projects')`. You might also notice that results come back faster.

There is one issue with this approach and that is that your schema file is not going to show this view. So if you want your CI to build correctly, you might have to change your schema option to generate sql instead. [It&#8217;s pretty simple to do this with Rails](http://edgeguides.rubyonrails.org/active_record_migrations.html#types-of-schema-dumps). The other caveat is that this introduces a cost to switching to a different database backend, because the SQL we now have in our migration is Postgres-specific.

Here are some good resources on views:

  1. <https://blog.pivotal.io/labs/labs/database-views-performance-rails>
  2. <http://blog.roberteshleman.com/2014/09/17/using-postgres-views-with-rails/> 
  3. [The Scenic gem for views, which gets around the schema problem](https://github.com/thoughtbot/scenic)

### In the application

If you don&#8217;t want to go the views route, you might consider organizing your queries with query objects and scopes. Have a look at [this](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/) for a brief introduction to query objects.

## Conclusion

With the information covered in this article, we can now begin to craft an answer for our hypothetical product manager, making the database server bear the brunt of the work.

**I&#8217;ve prepared a handy 3-page PDF cheatsheet containing the salient points of this article. If you&#8217;d prefer downloading and printing it out, [click here to get it!](/wp-content/uploads/2016/07/3_ways_to_work_time_in_postgres.pdf)**
