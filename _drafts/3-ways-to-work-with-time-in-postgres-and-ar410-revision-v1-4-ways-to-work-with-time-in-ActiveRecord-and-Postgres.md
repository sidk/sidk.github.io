---
id: 419
title: 4 ways to work with time in ActiveRecord and Postgres
date: 2016-07-21T20:43:40-04:00
author: Sid
layout: revision
guid: http://ducktypelabs.com/410-revision-v1/
permalink: /410-revision-v1/
---
Let&#8217;s say you&#8217;re developing a project management app with users and projects and the product manager asks you:

>   1. How much time does it take a given user to create their first project?
>   2. Give me a list of users who took longer than 1 month to create their first project.
>   3. What is the average amount of time that it takes for a user to create their first project?
>   4. What is the average amount of time per user between project creation?

Our models look like this:

    class User < ActiveRecord::Base
      has_many :projects
    end
    
    class Project < ActiveRecord::Base
      belongs_to :user
    end
    

As Rails developers, we tend to think in Ruby and use it as our first approach to problems like the above. Though that is fine, there can be performance issues if you constrain yourself to Ruby. As Rails developers, we also have access to a smart database server, whose capabilities we can leverage to attain performance benefits.

The SQL used in this article applies only to `Postgres`(version 9.4.1), though the ideas in general can be translated to other database management systems.

Before we go on, a brief primer on the `SELECT` function in SQL:

A `SELECT` statement retrieves zero or more rows from one or more database tables or database views.

So something like:

    SELECT * FROM users
    

will retrieve all available rows from the `users` table.

You can also pass in &#8220;aggregator&#8221; functions like MIN(), COUNT(), MEAN() etc to SELECT. When these functions are passed in to SELECT, the return value is a single piece of data (an integer, string, timestamp etc).

So for example this:

    SELECT MIN(created_at) FROM users
    

will return the earliest `created_at` (which is a timestamp) in the users table.

`SELECT` functions can also be nested within themselves, as we&#8217;ll see soon enough.

In `ActiveRecord` the [`select` method](http://api.rubyonrails.org/classes/ActiveRecord/QueryMethods.html#method-i-select) can be used to achieve the same result.

  1. Arithmetic operators (`-`, `+`, `*` and `/`)
    
    The first problem we have to tackle is figuring out how much time it takes a user to create their first project. This time equates to the difference between the user&#8217;s earliest `Project#created_at` and `User#created_at`.
    
    In Postgres, we can use the arithmetic difference operator `-` to calculate the difference between two timestamps or dates. The data type matters &#8211; if we calculate the difference between two timestamps, the return value is an &#8220;interval&#8221;, and if we calculate the difference between two dates, the return value is an integer representing the number of days between the two dates. More info <here>.
    
    To make it easy to display the data, we&#8217;ll plan to use the `select` method with something like:
    
        users = User.select('users.email, (projects.created_at - users.created_at) as time_to_first_project')
        
    
    This will return an `ActiveRecord::Relation` which we can then iterate over and call `#time_to_first_project` on:
    
        <% users.each do |user| %>
          <%= user.email %>
          <%= user.time_to_first_project %>
        <% end %>
        
    
    However, the above `select` won&#8217;t work for us for two reasons. One, ActiveRecord will complain that there is no `FROM` clause for the `projects` table. We can solve this with a call to `joins`:
    
        User.joins(:projects).select('users.email, (projects.created_at - users.created_at)....')
        
    
    Two, the above statement compares

The first piece of the puzzle is to ensure that only the user&#8217;s earliest project is used in the calculation. If we do:

    User.joins(:projects)
    
        users = User.joins(:projects)
                    .where('projects.created_at = (SELECT MIN(projects.created_at) FROM projects WHERE projects.user_id = users.id)')
                    .select("users.email, (projects.created_at - users.created_at) as time_to_first_project")
    

You can have `SELECT` statements nested within each other &#8211; in this case the nested statement is known as a &#8220;sub-query&#8221;

This will return a collection of `ActiveRecord` objects with the attributes `email` and `time_to_first_project`. You&#8217;ll notice that Postgres is pretty exact with its calculations and will return something like: `00:18:43.082321` or `9 days 12:48:48.220725`, which means you&#8217;ll have to do a bit of parsing and/or decorating before you present the information to the user.

The output of `-` depends on what you&#8217;re subtracting(https://www.postgresql.org/docs/9.1/static/functions-datetime.html). If you subtract two &#8220;dates&#8221; then the output will be an integer whose value represents the number of days between the two dates. If you subtract two &#8220;timestamps&#8221;, which in a default rails app is what columns like `created_at` and `updated_at` are, then the output is an &#8220;interval&#8221;. The &#8220;interval&#8221; has more detail.

If you have two timestamps and you want find the number of days between them, then you can use the CAST function to take advantage of the fact that when you subtract two dates you get the days between them.

  1. You can use AGE(..) instead of -.

  2. Just get the days with `EXTRACT` users = User.where(client: true) .joins(:projects) .where(&#8216;projects.created\_at = (SELECT MIN(projects.created\_at) FROM projects WHERE projects.user\_id = users.id)&#8217;) .select(&#8220;users.email, EXTRACT(DAY FROM projects.created\_at &#8211; users.created\_at) as time\_to\_first\_project&#8221;)

You can pass in a whole range of time based inputs to `EXTRACT`, like [`DAY`, `MONTH`, `YEAR`, `MILLISECOND`, etc](https://www.postgresql.org/docs/9.1/static/functions-datetime.html#FUNCTIONS-DATETIME-EXTRACT) You can also use `DATE_PART`.

So `EXTRACT` doesn&#8217;t work like you think it does. It only extracts the day &#8220;value&#8221;. So for example if an interval is 2 months 3 days, EXTRACT DAY FROM &#8230; will return 3(days).

  1. To compare the time passed to an arbitrary value: a. you can use the `INTERVAL` function. `INTERVAL` lets you filter your data based on a time input.  
    users.where(&#8220;(projects.created\_at &#8211; users.created\_at) > INTERVAL &#8217;60 days'&#8221;) &#8211; to filter out users.

`DATE`, `TIMESTAMP`, etc but doesn&#8217;t apply to above example.

What if you want calculate the average time between project creation for each user?

     User.where(client: true)
         .joins(:projects)
         .group('users.email')
         .having('COUNT(projects) > 1')
         .select("(CAST(MAX(projects.created_at) as date) - CAST(MIN(projects.created_at) as date))/(COUNT(projects) - 1) as avg_time_between_projects, users.email as email")
    

The above relies on the fact that the average difference between 4 sequential dates is the difference between the latest and earliest date divided by 3

`group` ensures the calculation is performed per user `having` is needed because `where` won&#8217;t work.

It can be beneficial to have this SQL reside in the DB instead of in your application layer. You can make this happen with views. As far as ActiveRecord is concerned, there is no difference between a view and a table. <what is a view?> The idea is that we&#8217;ll create a view with the user&#8217;s email and the fields we&#8217;re interested in, like average time between projects and time taken to create the first project. The benefit is that this will make our application code cleaner because we can now say something like `UserReport.where('average_time_between_projects > ?', 60)`

To create a view, first create a migration like so:

    sql = <<-SQL
    CREATE VIEW user_reports AS
      SELECT (CAST(MAX(packages.created_at) as date) - CAST(MIN(packages.created_at) as date))/(COUNT(packages)-1) as time_between_projects, COUNT(packages) as count, users.email as email FROM \"users\" INNER JOIN \"packages\" ON \"packages\".\"user_id\" = \"users\".\"id\" WHERE \"users\".\"role\" = 'client' GROUP BY users.email HAVING COUNT(packages) > 1    
      SQL
    execute(sql)
    

and then create a `UserReport` model class. Now you can do `UserReport.average('time_between_projects')` &#8211; you should notice that this is actually a lot quicker. (though I don&#8217;t fully understand the performance implications)

There is one issue with this approach and that is that your schema file is not going to show this view. So if you want your CI to build correctly, you might have to change your schema option to generate sql instead. [see this](http://edgeguides.rubyonrails.org/active_record_migrations.html#types-of-schema-dumps) for more info. The other caveat is that this introduces a cost to switching to a different database backend, because the SQL we now have in our migration is Postgres-specific.

how to create views: https://blog.pivotal.io/labs/labs/database-views-performance-rails how to create views: http://blog.roberteshleman.com/2014/09/17/using-postgres-views-with-rails/  
scenic gem for views, which gets around the schema problem: https://github.com/thoughtbot/scenic

To organize your code, use query objects.