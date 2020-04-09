---
id: 334
title: 'Freebie Offering Email 2: Dependency Injection'
date: 2016-05-13T12:33:17-04:00
author: Sid
layout: revision
guid: http://ducktypelabs.com/327-revision-v1/
permalink: /327-revision-v1/
---
Hi there,

This is part two of five of the Rails Testing and Technical Debt course you signed up for. Last time we discussed the importance of having tests before making a change to your codebase and how you can introduce tests to untested code in your app. By now, you should have an idea of where the inflection points in your code are. Today we&#8217;re going to be talking about how we can introduce tests to these inflection points, especially when our code is tightly coupled, by using a technique called &#8220;dependency injection&#8221;.

An object depends on another object if a change to either of them necessitates a change in the other. Dependencies, while essential for functioning in an OO system, can cripple your development efforts if left unchecked. There are four ways an object can depend on another object:

  * The object knows the name of another class
  * The object knows the name of another class&#8217;s method (or methods)
  * The object knows the arguments of another class&#8217;s method
  * The object knows the order of the arguments of another class&#8217;s method

In an ideal world, when you&#8217;re writing tests for a given class, the only object you&#8217;d need to initialize is the one under test. If you find yourself needing to set up and initialize a bunch of other objects or mocks just to get the test to run, this signals a dependency issue.

Consider a typical Rails controller, for example a `UsersController` with the familiar suite of RESTful actions. By convention, we&#8217;d expect the controller to at the very least know about the model it is tied to. But as it often happens, what if

Dependency injection is a technique through which When you begin to write tests for an inflection point (typically a class), you might find yourself having to set up a whole bunch of other objects. You might even need to set up more objects to ensure the functionality of these &#8216;other&#8217; objects. This is a clear sign that you have

avoid chaining prefer external dependencies over internal in Ruby (since external are essentially duck types)

Now that you&#8217;ve had a brief intro to authentication and the parts that comprise it, next time we&#8217;re going to talk about authorization.

All the best,

-Ali

P.S. Have any questions about authentication or Rails security in general? Let me know in a reply to this email and I&#8217;ll follow up as quickly as I can. Looking forward to hearing from you.