---
id: 92
title: How to use has_many :through with additional attributes on the join table
date: 2015-10-18T20:33:24-04:00
author: Sid
layout: post
guid: http://ducktypelabs.com/?p=92
permalink: /how-to-use-has_many-through-with-additional-attributes-on-the-join-table/
cta_content_placement:
  - below
classic-editor-remember:
  - classic-editor
categories:
  - ActiveRecord
---
A common area where developers can get hung up is when using `has_many :through`. `has_many :through` essentially allows you to link two models together with a &#8220;join table&#8221;. Though it can be pretty simple to work with, things can get confusing when you find yourself having to pass and store additional attributes in this join table.

> “I have read every post on the internet pertaining to has_many through with additional attributes on the join table and I am still not getting it&#8221;
> 
> “how do you store extra info in the join model and extract that out later?&#8221;

If you followed along [my previous post](http://ducktypelabs.com/how-a-has_many-through-association-works-in-practice/), at this point you should have a good idea of how to write your models, controllers and views such that you can create, display and edit records which use a `has_many :through` association. Make sure to check out [Ryan Bates screencast](http://railscasts.com/episodes/17-habtm-checkboxes-revised) on the basics, and also checkout [Rahoul&#8217;s article](http://theartandscienceofruby.com/2015/06/23/which-activerecord-association-should-i-use-has-and-belongs-to-many-or-has-many-through/) on when you should `has_many` in the first place.

So, what do you do when you want to store/access additional attributes in the join table?

You already know how to set up your view form so that it passes in parameters to your controller and correctly creates a new record in the join table. If you&#8217;re using checkboxes in your view, the checkbox code probably looks like this:

    <% Group.all.each do |group| %>
      <%= check_box_tag "user[group_ids][]", group.id, @user.group_ids.include?(group.id) %>
      <%= group.name %>
      <br />
    <% end %>
    

&#8230;assuming that you have a `User` and `Group` models and a `UserGroup` which functions as the join.

And your controller `#create` action should be very similar to this:

    def create
      User.create!(user_params)
    end
    
    def user_params
      params.require(:user).permit(group_ids: [])
    end
    

As you can see, we don&#8217;t have to do anything special in the controller because `group_ids` will be accepted as a parameter (it is part of the dynamic programming which happens when you call `has_many`). You do have to correctly permit the `group_ids` param though.

Now, let&#8217;s say for a given user you want to specify if they are an admin of the group or not, via checkbox.

I&#8217;m going to assume that we will have an edit page for each group record, and on this page, we will see all the users that belong to this group along with a checkbox next to each user indicating if they are an admin or not.

### The View

In my group edit page, I want to show the users that are in the group and next to each user show a checkbox allowing me to select if the user is going to be an admin.

    <%= form_for @group do |f| %>
      <%= @group.inspect %>
      <br />
      <% if @group.user_groups.present? %>
        <%= f.fields_for :user_groups do |ugf| %>
          <% user = ugf.object.user %>
          <%= user.name %>
          Admin?
          <%= ugf.check_box :admin %>
          <br />
        <% end %>
      <% else %>
        No Users in this group yet
      <% end %>
      <%= f.submit 'Update group' %>
    <% end %>
    

Couple of things to note here:

1) I&#8217;m using the `user_groups` association. By using `fields_for` on this association, I can treat it like any other association of `group` and build a custom form for it.

2) When submitting this form, the `user_groups` parameters will be passed in under the `user_groups_attributes` key in the `params` hash.

### The Model

To be able to pass in a hash with `user_groups_attributes` to the `Group` model and call `update` or `save` on it, we need to use the `accepts_nested_attributes_for` method. This method tells Rails and ActiveRecord how to correctly deal with `user_groups_attributes` being in the `params` hash.

Our `Group` would look like this:

    class Group < ActiveRecord::Base
      has_many :user_groups
      has_many :users, through: :groups
      accepts_nested_attributes_for :user_groups
    end
    

So the parameter we pass in to `accepts_nested_attributes_for` is the model/association we want to accept nested attributes for.

### The Controller

Because of the setup we did above in the `Group` model, we can now use the usual `update` method with the params that we get from the view/form.

    def update
      @group = Group.find(params[:id])
      @group.update(group_params)
    end
    
    def group_params
      params.require(:group).permit(user_groups_attributes: [:admin, :id])
    end
    

If you&#8217;re using Rails 4 and `strong_parameters`, you will have to make sure you permit the correct parameters.

And that&#8217;s it! You should now be able to update this admin attribute on the `UserGroup` join table. You can follow the same approach for different types of data as well (like a text field for example). I encourage you to look deeper into what the params hash looks like once it gets to the controller so that you get more comfortable with it. Play with this idea in Rails console as well to increase your confidence.

I&#8217;ve posted an [example app on github](https://github.com/sidk/has_many_through_with_additional_attributes) &#8211; check it out if you need more info about how exactly to get this to work.
