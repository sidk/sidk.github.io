---
id: 176
title: All About Foreign Keys, References and the .joins method
date: 2016-01-08T18:11:51-05:00
author: Sid
layout: revision
guid: http://ducktypelabs.com/162-revision-v1/
permalink: /162-revision-v1/
---
After working for a while on Rails and ActiveRecord, you will probably reach a point (like I did) where you want to understand more about how ActiveRecord works with your database. Terms like &#8220;foreign keys&#8221;, &#8220;references&#8221;, &#8220;joins&#8221;, &#8220;associations&#8221; etc will probably make sense to you on an intuitive level, but to build efficient and elegant web applications with Ruby and Rails, it will benefit you greatly to deepen your understanding of the differences between these concepts and have an idea when one should or should not be used.

### Foreign Keys

A foreign key is a mechanism that you can use to associate one table to another. Typically, it takes the form of a column with the name `<other_table_name>_id`. Let&#8217;s say you have a `User` table and a `Project` table, and you&#8217;ve used a `has_many` on `User`, you&#8217;ll see that a statement like `user.projects` when translated into SQL by ActiveRecord will automatically use `user_id` as a foreign key. Check it out yourself and see, try:

    user.projects.to_sql
    

in your console. You should see an SQL statement that is searching for all projects with a `user_id` corresponding to `user`&#8216;s ID.

In other words, ActiveRecord relies on this column to figure out which projects of all in the `projects` table are &#8220;related&#8221; or &#8220;associated&#8221; to the given user.

You probably know that doing `has_many` is not enough. You will also have to add a column to the `Project` table with the name `user_id`. Your migration might look like:

    add_column :projects, :user_id, :integer
    

Newer versions of ActiveRecord also provide the `#references` and `#add_reference` method, which take care of adding the foreign key column for you. Modifying the example code above, instead of using the `add_column` method, you can use the `add_reference` method, like so:

    add_reference :projects, :user
    

This will add a `user_id` column to the `projects` table. The advantage with using the `reference` method is that you can add an index and a database constraint on the foreign key painlessly as well.

Check out [this doc](http://guides.rubyonrails.org/association_basics.html) and [this doc](http://edgeguides.rubyonrails.org/active_record_migrations.html) for more info.

#### Referential Integrity

If you were wondering what I meant when I said &#8216;database constraint on the foreign key&#8217;, rest assured, because I&#8217;m going to cover what that means here.

It depends on who you&#8217;re talking to, but in many contexts, defining a foreign key in the Rails world is understood plainly as creating or adding a foreign key column as described above. The problem with this is that when we only have a foreign key column, we have no way of ensuring that for a given value of `user_id`, a corresponding `User` record actually exists. This has the potential to lead to confusion down the line.

To know what I mean, fire up your console and try the following (assuming you have the same setup as above, and that we have one `User` in the system with `ID = 1`)

    project = Project.create(name: 'hit song', user_id: 2)
    

You should be able to create the project, but when you do `project.user`, you&#8217;ll either get an error or get back nil. You can of course use a `validates` in your model to ensure uniqueness and/or presence of the `user`, but

Mention `add_foreign_key` migration and how this enforces referential integrity. Mention that you want to index, and why you should index

Why it is important

The `add_foreign_key` method

https://robots.thoughtbot.com/referential-integrity-with-foreign-keys

http://edgeguides.rubyonrails.org/active\_record\_migrations.html

http://guides.rubyonrails.org/association_basics.html (do a search for &#8220;references&#8221;)