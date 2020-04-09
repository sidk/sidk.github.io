---
id: 411
title: Time functions in SQL
date: 2016-07-07T17:32:50-04:00
author: Sid
layout: revision
guid: http://ducktypelabs.com/410-revision-v1/
permalink: /410-revision-v1/
---
Primer about the `SELECT` function. Explain `SELECT MIN(...) FROM ... WHERE ...`

A `SELECT` statement retrieves zero or more rows from one or more database tables or database views.

You can have `SELECT` statements nested within each other &#8211; in this case the nested statement is known as a &#8220;sub-query&#8221;

The task is: give me a list of clients who took x amount of time between event A and event B

event A & B happen in separate tables, which are associated with the User table

for example: let&#8217;s say you&#8217;re running a SaaS like basecamp and your boss asks you:

give me a list of clients who took longer than 1 month to create their first project  
what is the average amount of time that it takes for a client to create their first project  
on a week to week basis, how many projects are being started, and how many clients are joining?  
make this into a live report  
Options:

You can do it in Ruby &#8211; this can take a long time depending on how many users you have  
You can do it in SQL &#8211; major performance benefits, because it makes the DB do most of the work. DB is not so dumb.  
We&#8217;re going to talk about how to do it with SQL:

  1. Plain old `-` and `+`.
    
        users = User.where(role: 'client')
                    .joins(:packages)
                    .where('packages.created_at = (SELECT MIN(packages.created_at) FROM packages WHERE packages.user_id = users.id)')
                    .joins(meal_selection: [:service_days])
                    .where('service_days.updated_at = (SELECT MIN(service_days.updated_at) FROM service_days WHERE service_days.meal_selection_id = meal_selections.id)')
                    .select("users.*, (service_days.updated_at - packages.created_at) as time_to_order")
        

Just get the days with `DATE_PART`

    users = User.where(role: 'client')
                .joins(:packages)
                .where('packages.created_at = (SELECT MIN(packages.created_at) FROM packages WHERE packages.user_id = users.id)')
                .joins(meal_selection: [:service_days])
                .where('service_days.updated_at = (SELECT MIN(service_days.updated_at) FROM service_days WHERE service_days.meal_selection_id = meal_selections.id)')
                .select("users.*, DATE_PART(‘day', service_days.updated_at - packages.created_at) as time_to_order")
    

And then:

    users.select { |u| u.time_to_order.to_i >= <X Days> }.map { |u| p [u.email, u.time_to_order] }
    

Can we use Arel for this?