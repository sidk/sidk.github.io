---
id: 320
title: 'Freebie Offering Email 1: Why Tests are Essential for Better Code'
date: 2016-05-13T12:54:20-04:00
author: Sid
layout: page
guid: http://ducktypelabs.com/?page_id=320
cta_content_placement:
  - below
---
Hi there,

Thanks for signing up to receive my five-part email course on reducing technical debt in your Rails app. This is part one of five of the course.

I&#8217;m Sid Krishnan, web developer and founder of a little consultancy called Duck Type Labs. My goal for this course is to help you get a handle on technical debt in your Rails app.

You know you have technical debt in your system. You also know that to keep pushing features out the door to waiting customers, you have to get a handle on your technical debt so that &#8220;interest payments&#8221; don&#8217;t end up crippling new feature development. Thirdly, you know that to reduce technical debt in your application, you have to refactor your code such that adding a new feature or fixing a bug is easier.

However, refactoring a codebase is not without its challenges. It might involve touching large areas of the codebase. Or you might find yourself needing to experiment with different ways of organizing the codebase to align with terms of art and ways of operation in your business. You need to be able to do these things without fearing that you or your teammates will introduce subtle bugs in your application.

If you have good test coverage of the areas you want to change and/or refactor, you&#8217;re in a good spot. You can go ahead and make your changes with impunity, just as long your tests pass for every change you make, and you make sure you keep test coverage up by writing new tests. But what if your test coverage is spotty at best, or even worse, you don&#8217;t have tests at all?

Note: If you haven&#8217;t already, I recommend running your codebase through a test coverage tool like `simplecov`. This will give you a good idea of how far you have to go to get your code to a place where it can be refactored.

Michael Feathers, author of the book &#8220;Working Effectively with Legacy Code&#8221;, recommends the following strategy to cover untested code with tests:

  1. Identify change points
  2. Find an inflection point
  3. Cover the inflection point a. Break external dependencies b. Break internal dependencies c. Write tests
  4. Make changes
  5. Refactor the covered code.

&#8220;Change Points&#8221; are the areas in your codebase that you need to change to accomplish your objective &#8211; things like adding a new feature, fixing a bug, or maybe a refactor to make things clearer. Making a change, depending on how your code is structured, can be easy or hard. Typically, the more change points there are, the harder the change will be. When you&#8217;re touching many areas of the codebase, you&#8217;re more likely to cause unintended consequences. If there&#8217;s more than one way to accomplish a change, and you don&#8217;t have tests, it will probably be easier to choose the path with the least amount of change, because you&#8217;ll have to write fewer tests.

Your goal: introduce just enough coverage so that any changes you make will be caught by your tests. The rest of your application will typically interface with a class or a set of classes to send and/or receive information contained in your change points. This class or set of classes is known as an &#8220;Inflection Point&#8221; or &#8220;Test Point&#8221;. More often than not, the inflection point is not the same as the change point.

A modification to a change point can only affect application behavior through the inflection point. Put another way, the rest of your application receives state changes through methods on the inflection point. Essentially, comprehensive test coverage at your inflection point will give you enough confidence to make the changes you need without causing unintentional side-effects. For a rather simplistic example, recall how when you&#8217;re testing a Ruby class, you don&#8217;t bother testing the private methods because you can be reasonably sure that changes made to them will make themselves felt in tests for the public methods.

Summary/Action Items:

  1. Before you begin to make a change to your system, whether its adding a feature, fixing a bug, or removing technical debt, you need to ensure a reasonable amount of test coverage. When your code is covered with tests, you can make changes and be confident that the changes you make don&#8217;t cause subtle breakages.

  2. To introduce test coverage to an untested codebase, you need to first: a. Identify change points &#8211; the areas of your codebase you need to change to accomplish your objective b. Identify the inflection point &#8211; a narrow interface which the rest of your application uses to interface with your change points.

  3. A tool like [&#8216;simplecov&#8217;](https://github.com/colszowka/simplecov) can give you a good idea of which areas of your codebase lack test coverage. Aim for above 90% coverage. I&#8217;d also recommend setting up filters to exclude your &#8216;/spec&#8217; directory and groups for &#8216;/models&#8217;, &#8216;/controllers&#8217;, &#8216;/views&#8217;, &#8216;/lib&#8217; and any other directories you&#8217;re interested in.

After you&#8217;ve identified inflection points, the next step is to write tests for them. This can be a non-trivial task depending on how tightly coupled your code is. Next time, we&#8217;ll take an introductory look at how the way your tests are structured can indicate problems with your design.

Until then,

-Sid

P.S. Have questions about testing and technical debt in your Rails app that you need answered? Now that you&#8217;re on the course, don&#8217;t hesitate to get in touch with me directly. Just reply to this email with any questions and I&#8217;ll try to get back to you as soon as possible.