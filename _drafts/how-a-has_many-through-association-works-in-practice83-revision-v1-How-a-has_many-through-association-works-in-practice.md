---
id: 481
title: How a has_many :through association works in practice
date: 2017-05-26T17:15:59-04:00
author: Sid
layout: revision
guid: https://ducktypelabs.com/83-revision-v1/
permalink: /83-revision-v1/
---
When you&#8217;re beginning to work with Rails and ActiveRecord, it can be tough to wrap your head around the concept of has_many :through associations, and more importantly how they can be used in your Rails app.

> “lets say i have group, member, memberships&#8230;how do i then actually USE this in my models, controllers and views?&#8221;
> 
> &#8220;I&#8217;m using a has_many :through relationship which appears to be working. I now need a form that I can render to have users join or leave the group.&#8221;

First off, you should be reasonably confident that a has_many :through association is a good choice for your app design. Assuming you are, here&#8217;s how you can put it into practice.

Let&#8217;s say you&#8217;re building an app with Users and Groups, and you want to build a form where a given user can join or leave any group. You&#8217;ve chosen to associate your users and groups through the table Membership.

Your models are structured like so:

    class User < ActiveRecord::Base
      has_many :memberships
      has_many :groups, through: :memberships
    end
    
    class Group < ActiveRecord::Base
      has_many :memberships
      has_many :users, through: :memberships
    end
    
    class Membership < ActiveRecord::Base
      belongs_to :user
      belongs_to :group
    end
    

I always find it helpful before I start writing my controllers or views, to think through what I want to accomplish, and use the Rails console to ensure I&#8217;m on the right path. In other words, I try to come up a with an end-goal, and then work backwards with the help of the Rails console to figure out my solution.

So, given that we want to render a form that allows a user to join or leave a group, we should ask ourselves, &#8220;what does it mean to have a user _join_ a group?&#8221;

As far as the database is concerned, it essentially means that the `Membership` table will have a row with the `user_id` and `group_id` columns corresponding to our given user and group. Our task then is to orchestrate things such when the user clicks &#8216;submit&#8217; on their form, the `Membership` table ends up with a new row (or rows) with the correct `user_id` and `group_id` values.

In the form/view that we render for the user, we need a way whereby the user can select which groups they want to join. And in the controller, we want to extract the choices that the user made in the form and use those choices to create new rows in the `Membership` table.

First, imagine you had a `user_id` and a `group_id`, how would you go about creating that new row in the Membership table? ActiveRecord makes it easy. What we want to do is, for a given `User` record, add a &#8216;membership&#8217; to it, so that when we say `@user.groups`, we get back a list of groups which includes the group we just added. If you do this:

    #assuming @user is your User record
    @user.groups << Group.find(group_id)
    
    #if you have a list of group_ids(something like [1, 2]), you can also do:
    @user.group_ids << group_ids
    

This piece of code will automagically create a new row in the Membership table with the right `group_id` and `user_id`. And now whenever you use `@user.groups`, you will get back a list of groups that you added. For added confidence, try the above in your Rails console.

Check out the Rails guide below for [more details](http://guides.rubyonrails.org/association_basics.html#the-has-many-through-association) on what methods ActiveRecord provides you when you use `has_many :through`.

I&#8217;m going to assume you have some type of authentication set up in your app, which gives you access to the `current_user` method. We will need this method to get our `user_id`, via a simple `current_user.id` call. Once you have your user (with something like `@user = User.find(current_user.id)`), you can use the code above to add groups.

The next question is, how do you write your view such that it passes the `group_id` values to the controller?

How you write your view will depend wholly on the flow you expect your users to go through and other factors in your app. Let&#8217;s say you decide to provide your user with a list of checkboxes listing the groups they can join or leave. Here&#8217;s a simple way to accomplish that:

    <% Group.all.each do |group| %>
      <%= check_box_tag "user[group_ids][]", group.id, @user.group_ids.include?(group.id) %>
      <%= group.name %>
      <br />
    <% end %>
    

The usage of `check_box_tag` probably needs some explanation.

The first parameter `"user[group_ids][]"` tells Rails to group all the checkboxes with this ID together and pass it in to the controller as an Array. What this means is in your controller, you can do `params[:user][:group_ids]` and get a nice Ruby Array of all the `group_ids` that the user chose.

The last parameter just ensures that any Group which the user currently belongs to is checked when the form is rendered.

The above should get you going with using has_many :through in your Rails app, but depending on what your app does I&#8217;m pretty sure you will run into issues which are not covered in this post. So if you&#8217;re still stuck, check out the following:

1) Check out [this amazing Rails cast](http://railscasts.com/episodes/17-habtm-checkboxes-revised) which takes you through the entire gamut of using has_many :through. If you haven&#8217;t already, I&#8217;d highly recommend you get yourself an account on RailsCasts.

2) [Rails ActiveRecord Guide on has_many :through](http://guides.rubyonrails.org/association_basics.html#the-has-many-through-association)

If you need more help, hit reply in the comment section below, and I&#8217;ll do my best to get you going.

<div id="mc_embed_signup" style="background: #dff7fe; padding: 15px;">
</div>