---
id: 417
title: 4 ways to parse time in Postgres
date: 2016-07-19T12:42:41-04:00
author: Sid
layout: revision
guid: http://ducktypelabs.com/410-revision-v1/
permalink: /410-revision-v1/
---
Primer about the `SELECT` function. Explain `SELECT MIN(...) FROM ... WHERE ...`

A `SELECT` statement retrieves zero or more rows from one or more database tables or database views.

You can have `SELECT` statements nested within each other &#8211; in this case the nested statement is known as a &#8220;sub-query&#8221;

You can also pass in &#8220;aggregator&#8221; functions like MIN(), COUNT(), MEAN() etc to SELECT.

The task is: give me a list of clients who took x amount of time between event A and event B

event A & B happen in separate tables, which are associated with the User table

for example: let&#8217;s say you&#8217;re running a SaaS like basecamp and your boss asks you:

give me a list of clients who took longer than 1 month to create their first project what is the average amount of time that it takes for a client to create their first project on a week to week basis, how many projects are being started, and how many clients are joining? make this into a live report Options:

You can do it in Ruby &#8211; this can take a long time depending on how many users you have You can do it in SQL &#8211; major performance benefits, because it makes the DB do most of the work. DB is not so dumb. We&#8217;re going to talk about how to do it with SQL:

  1. Plain old `-` and `+`.
    
        users = User.where(client: true)
                    .joins(:projects)
                    .where('projects.created_at = (SELECT MIN(projects.created_at) FROM projects WHERE projects.user_id = users.id)')
                    .select("users.email, (projects.created_at - users.created_at) as time_to_first_project")
        

This will return a collection of `ActiveRecord` objects with the attributes `email` and `time_to_first_project`. You&#8217;ll notice that Postgres is pretty exact with its calculations and will return something like: `00:18:43.082321` or `9 days 12:48:48.220725`, which means you&#8217;ll have to do a bit of parsing and/or decorating before you present the information to the user.

  1. Just get the days with `EXTRACT`
    
    users = User.where(role: &#8216;client&#8217;)  
    .joins(:packages)  
    .where(&#8216;packages.created\_at = (SELECT MIN(packages.created\_at) FROM packages WHERE packages.user_id = users.id)&#8217;)  
    .joins(meal\_selection: [:service\_days])  
    .where(&#8216;service\_days.updated\_at = (SELECT MIN(service\_days.updated\_at) FROM service\_days WHERE service\_days.meal\_selection\_id = meal_selections.id)&#8217;)  
    .select(&#8220;users.*, EXTRACT(DAY FROM service\_days.updated\_at &#8211; packages.created\_at) as time\_to_order&#8221;)

You can pass in a whole range of time based inputs to `EXTRACT`, like [`DAY`, `MONTH`, `YEAR`, `MILLISECOND`, etc](https://www.postgresql.org/docs/9.1/static/functions-datetime.html#FUNCTIONS-DATETIME-EXTRACT) You can also use `DATE_PART`.

  1. users.where(&#8220;(service\_days.updated\_at &#8211; packages.created_at) > INTERVAL &#8217;60 days'&#8221;) &#8211; to filter out users.

And then:

    users.select { |u| u.time_to_order.to_i >= <X Days> }.map { |u| p [u.email, u.time_to_order] }
    

Can we use Arel for this?