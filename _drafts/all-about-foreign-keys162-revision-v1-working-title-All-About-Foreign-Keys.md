---
id: 183
title: (working title) All About Foreign Keys
date: 2016-01-10T13:20:36-05:00
author: Sid
layout: revision
guid: http://ducktypelabs.com/162-revision-v1/
permalink: /162-revision-v1/
---
After working for a while with Rails and ActiveRecord, you will reach a point (like I did) where you want to understand more about how ActiveRecord works with your database. Terms like &#8220;foreign keys&#8221;, &#8220;references&#8221;, &#8220;joins&#8221;, etc will probably make sense to you on an intuitive level as you&#8217;ve become familiar with the ActiveRecord API. However, to build efficient and elegant web applications with Ruby and Rails, it will benefit you greatly to deepen your understanding of the differences between these concepts and have a grasp on when one should or should not be used. Of these terms, the concept of foreign keys has quite the potential to confuse.

### Foreign Keys

A foreign key is a mechanism that you can use to associate one table to another. Typically, it takes the form of a column with the name `<other_table_name>_id`. Let&#8217;s say you have a `User` table and a `Project` table, and you&#8217;ve used a `has_many` on `User`, you&#8217;ll see that a statement like `user.projects` when translated into SQL by ActiveRecord will automatically use `user_id` as a foreign key. Check it out yourself and see, try:

    user.projects.to_sql
    

in your console. You should see an SQL statement that is searching for all projects with a `user_id` corresponding to `user`&#8216;s ID.

ActiveRecord relies on this column to figure out which projects of all in the `projects` table are &#8220;related&#8221; or &#8220;associated&#8221; to the given user.

You probably know that doing `has_many` is not enough. You will also have to add a column to the `Project` table with the name `user_id`. Your migration might look like:

    add_column :projects, :user_id, :integer
    

Newer versions of ActiveRecord also provide the `#references` and `#add_reference` method, which take care of adding the foreign key column for you. So, instead of using the `add_column` method, you can use the `add_reference` method, like so:

    add_reference :projects, :user
    

This will add a `user_id` column to the `projects` table. The advantage with using the `reference` method is that you can add an index and a database constraint on the foreign key painlessly as well.

Check out [this doc](http://guides.rubyonrails.org/association_basics.html) and [this doc](http://edgeguides.rubyonrails.org/active_record_migrations.html) for more info.

### Referential Integrity

If you were wondering what I meant when I said &#8216;database constraint on the foreign key&#8217;, rest assured, because I&#8217;m going to cover what that means here.

It depends on who you&#8217;re talking to, but in many contexts, a foreign key in the Rails world is understood plainly as the column itself. Depending on your use case, this could be enough. Keep in mind though, that the problem with this approach is that when we only have a foreign key column, we have no way of ensuring that for a given value of `user_id`, a corresponding `User` record actually exists. This has the potential to lead to confusion down the line.

To know what I mean, fire up your console and try the following (assuming you have the same setup as above, and that we have one `User` in the system with `ID = 1`)

    project = Project.create(name: 'hit song', user_id: 2)
    

You should be able to create the project, but when you do `project.user`, you&#8217;ll get back `nil`. AKA an _orphaned_ project. You can solve this with a `validates` in your model to ensure uniqueness and/or presence of the `user`, but you might find yourself in a situation where you&#8217;re bypassing validation callbacks somehow (probably with a bulk operation), and yet again creating orphans. What solves this problem, is getting the _database_ to ensure that orphaned records are not created.

Enter the `add_foreign_key` method, which you&#8217;ll notice is sort of a second meaning for the term &#8216;foreign key&#8217;. Using this method in your migration tells the database (in the way only ActiveRecord knows how) to enforce _referential integrity_ by adding a _foreign key constraint_. [This very good article from the robots at thoughtbot](https://robots.thoughtbot.com/referential-integrity-with-foreign-keys) goes into many of the finer details of this concept, and why it&#8217;s important. You&#8217;d use it like so:

    add_column :projects, :user_id
    add_foreign_key :projects, :users
    

[This Rails doc](http://edgeguides.rubyonrails.org/active_record_migrations.html#foreign-keys) gives some additional details.

If you&#8217;re using the `add_reference` method, you can pass in a boolean to indicate if a foreign key constraint is to be added, like so:

    add_reference :projects, :user, foreign_key: true
    

### The `.joins` method

If you came to Rails without a background in SQL (like me), you&#8217;ve probably wondered about the `.joins` method, how it is related to the concept of associations and foreign keys, and when you should use it.

The rule of thumb with the `.joins` method is that you need to use it when you need to query more than one table at the same time. In other words, if the query you&#8217;re writing for a given table/model needs to reference attributes in another table (or more), you will need to invoke `.joins` to make this happen. This holds true _regardless of if the tables are associated to each other with a foreign key column or constraint_.

So for example, if we want to query the `projects` table for all projects that are owned by a user of role &#8216;manager&#8217; (assuming we have a &#8216;role&#8217; column on the `users` table), we&#8217;d do:

    Project.joins(:user).where(user: { role: 'manager' })
    

You&#8217;ll notice that if you try this without the `.joins`, ActiveRecord will complain.

For some background on joins, check out this [nice summary on the different types of joins you can do in SQL.](http://www.sql-join.com/) You can even use the `.joins` method to query multiple tables that are not associated to one another if you so choose. I encourage you to fire up the console and use the `.to_sql` method for your queries whenever you&#8217;re stuck to get a better handle on what ActiveRecord is doing behind the scenes.

Hope this helped, and as always, please leave a comment below if you&#8217;re stuck on something that you need help with or if something above was unclear in any way. I&#8217;d love to help out! [mc4wp_form]