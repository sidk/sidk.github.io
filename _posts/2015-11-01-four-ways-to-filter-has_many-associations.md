---
id: 100
title: 4 ways to filter has_many associations
date: 2015-11-01T16:23:56-05:00
author: Sid
layout: post
guid: http://ducktypelabs.com/?p=100
permalink: /four-ways-to-filter-has_many-associations/
cta_content_placement:
  - below
classic-editor-remember:
  - classic-editor
categories:
  - ActiveRecord
---
You have a model that `has_many` associations, and you&#8217;d like to get a collection of these models with their associations that satisfies a list of conditions both on the model and the association records. You&#8217;ve tried a few things, but whatever you&#8217;ve tried only &#8220;sort of&#8221; works (meaning, not really) and leaves you feeling like you need a push in the right direction.

Filtering `has_many` associations can be one of those problems which makes you scratch your head and want to do some extra reading to brush up on your `SQL` and `ActiveRecord` knowledge.

For example, let&#8217;s say you have `User` and `Project` models in your system, and you want to write a query that retrieves a collection of `users` with `projects` that were created in a given date range.

Depending on what you want your query to return, here are 4 different ways to approach this problem:

  1. The simplest thing you can do is to combine the `joins` and `where` methods that `ActiveRecord` gives you, like so:
    
        User.joins(:projects).where(projects: { zipcode: 30332 })
        
    
    This technique is a good choice when you need to filter your records by one or more attributes. This will give you back a collection of `User` records which all have projects with a zipcode of 30332.
    
    One gotcha to keep in mind is that since you&#8217;re doing a `INNER JOIN` with the `joins` method, **this can return duplicates**. Read on to point 2 to see how you can get around this.

  2. More often than not, you might have scopes defined on your association models which you want to re-use. So when you want to use these scopes to do your filtering, get acquainted with `ActiveRecord's` `merge` function. According to the [`ActiveRecord` documentation](http://apidock.com/rails/ActiveRecord/SpawnMethods/merge), `merge` returns an array representing the intersection of the `ActiveRecord::Relation` that you call `merge` on, and the `ActiveRecord::Relation` that you pass in.
    
    So, for example, if you have a scope in your `Project` model called `opened_recently`, which returns all `projects` which have been created within the last 10 days, you can do something like:
    
        User.joins(:projects).merge(Project.opened_recently)
        
    
    This will return to you a list of `User` objects which all satisfy the condition of having projects which were opened recently.
    
    Like I said before, one thing to keep in mind when you&#8217;re using `joins` for a `has_many` association is that since you are doing an `INNER JOIN`, **it is possible that you will get back duplicate `User` (in our example) objects** &#8211; basically one object for every &#8216;match&#8217;. It&#8217;s easy enough to get around this with a call to the `uniq` method.
    
        User.joins(:projects).merge(Project.opened_recently).uniq
        
    
    The `uniq` method works by changing your query from something like ``SELECT `users` from ...`` to something like ``SELECT DISTINCT `users` from...``

  3. If you&#8217;re going to be needing to retrieve information from the records you filter, especially information stored in the associations of these records, another option to consider is [eager loading and the `includes` method](http://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations). As you probably know, the benefit of eager loading is that you get to avoid the N + 1 queries problem.
    
    It&#8217;s pretty straightforward to filter by attributes or scopes on the association:
    
        User.includes(:projects).where(projects: { zipcode: '30332' })
        
    
    This will give us all `users` who have projects in Atlanta, and also eager load their projects.
    
    If in your `.where` you want you use an SQL fragment, then you need to call the [`references`](http://apidock.com/rails/ActiveRecord/QueryMethods/references) method:
    
        User.includes(:projects).where('projects.deleted_at IS NOT NULL').references(:projects)
        
    
    You can also use the `merge` method in conjunction with `includes` and `references` to make use of any scopes you might have defined in your association models, like so:
    
        User.includes(:projects).merge(Project.opened_recently).references(:projects)
        
    
    You won&#8217;t need to use `uniq` when you use `includes` because `ActiveRecord` will take care of it for you automatically.

<div id="mc_embed_signup" style="background: #dff7fe; padding: 15px;">
</div>

  1. Now, what should you do if you want your collection to only have those associations which meet your filter criteria? This situation arose for me when I had to write an API endpoint which let app users pass in parameters to filter associations. There are a couple of ways you can approach this. In my case, since I had to return an object that would eventually be converted to JSON, the simplest way was to manually construct a hash which contained the record and the filtered associations.

So I did something like this in my controller:

    def show 
      user_attributes.merge(projects: filtered_projects) 
    end
    
    
    def user_attributes 
      # returns a hash containing attributes for the given user 
      # something like this: # { id: 1, first_name: 'Rick', last_name: 'Sanchez' } 
    end
    
    def filtered_projects 
      # here's where I used any Project related params to filter down my projects 
      # something like Project.opened_after(params[:project_date]).as_json 
      # I use as_json because I need this method to return a hash 
    end
    

Note that the `merge` method used here is the one for `Hash`, and not the `ActiveRecord` method. You should be able to extend this to return multiple users as well.

Another way to do this is by defining a scoped `has_many` association. If you&#8217;re interested in seeing how this might be done, ping me in the comment section below and I&#8217;ll do a post on this topic specifically.

Hope this was helpful. As always, if you have a question about this topic that isn&#8217;t specifically addressed above, leave a comment below, and I&#8217;ll do my best to help out!

<div id="mc_embed_signup" style="background: #dff7fe; padding: 15px;">
</div>