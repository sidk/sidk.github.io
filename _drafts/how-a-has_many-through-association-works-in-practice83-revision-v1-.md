---
id: 85
date: 2015-10-07T14:01:16-04:00
author: Sid
layout: revision
guid: http://ducktypelabs.com/83-revision-v1/
permalink: /83-revision-v1/
---
## How a has_many :through association works in practice

Proof/Supporting Detail &#8211; proof that I understand the problem; Quotes work here too:

When you&#8217;re beginning to work with Rails and ActiveRecord, it can be tough to wrap your head around the concept of has_many :through associations, and more importantly how they can be used in your Rails app.

> “lets say i have group, member, memberships&#8230;how do i then actually USE this in my models, controllers and views?&#8221;
> 
> &#8220;I&#8217;m using a has_many :through relationship which appears to be working. I now need a form that I can render to have users join or leave the group.&#8221;

Answer: Just one answer

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

CTA: