---
id: 63
date: 2015-09-30T14:07:40-04:00
author: Sid
layout: revision
guid: http://ducktypelabs.com/60-revision-v1/
permalink: /60-revision-v1/
---
Title: Retrieving Random records in Rails

Question: What is the best way to retrieve one or more random records in Rails?

<div>
</div>

<div>
  Given that there are multiple ways to retrieve random records in Rails and Ruby (take a look at this SO post &#8211; http://stackoverflow.com/questions/5297396/quick-random-row-selection-in-postgres), it can be confusing and frustrating to decide which course of action to take.
</div>

<div>
</div>

<div>
  &#8220;Is there a technique to retrieve random records that is correct/general/Rails way/cross-database compatible? &#8220;
</div>

<div>
</div>

<div>
  In this post, I&#8217;ll cover a couple of simple approaches that you can take to figure out what the best way might be for you.
</div>

<div>
</div>

<div>
  Answer:
</div>

<div>
</div>

<div>
  The first priority that you likely have to keep in mind is speed. Most applications have a threshold for what is considered slow &#8211; in web apps for example, anything that takes longer than a few hundred milliseconds to load is considered slow. So you don&#8217;t want to be choosing a method that will have your users waiting around forever for your app to do something. That being said, there is also no point in sacrificing code readability, ease of re-use and just time in general, to get faster than you have to.
</div>

<div>
</div>

<div>
  Take this common solution to retrieve a random record in Rails:
</div>

<div>
</div>

<pre lang="RUBY" line="1">random_user = User.all.sample</pre>

<div>
</div>

<div>
  CTA:
</div>

<pre lang="RUBY" line="1">Fix:

  def to_s
    "#{name} - #{county_name}, #{location_name}"
  end

  def state_id_name
    "#{location_state_abbr}-#{id}-#{name}"
  end

  def state_name
    return location_name if location_type == 'State'
    location.state.name
  end
</pre>