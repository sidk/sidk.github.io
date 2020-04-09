---
id: 337
title: 'Freebie Offering Email 2: Ugly Tests'
date: 2016-05-13T14:36:15-04:00
author: Sid
layout: revision
guid: http://ducktypelabs.com/327-revision-v1/
permalink: /327-revision-v1/
---
Hi there,

This is part two of five of the Rails Technical Debt course you signed up for. Last time we discussed the importance of having tests before making a change to your codebase and how you can introduce tests to untested code in your app. By now, you should have an idea of where the inflection points in your code are, and begun to write tests for them. Today we&#8217;re going to be talking about how tests can signal to us if our code is too tightly coupled.

An object depends on another object if a change to either of them necessitates a change in the other. Dependencies, while essential for functioning in an OO system, can cripple your development efforts if left unchecked. There are four ways an object can depend on another object:

  * The object knows the name of another class
  * The object knows the name of another class&#8217;s method (or methods)
  * The object knows the arguments of another class&#8217;s method
  * The object knows the order of the arguments of another class&#8217;s method

When a given object knows too much about other classes, or worse yet, about classes that other classes know about (for example something like `user.menu.meals.first`), this object is at high risk for change when any one of those classes change.

In an ideal world, when you&#8217;re writing tests for a given class, the only object you&#8217;d need to initialize is the one under test. If you find yourself needing to set up and initialize a bunch of other objects or mocks (especially if they&#8217;re deeply nested) just to get the test to run, this signals a dependency issue, and the fact that you&#8217;d benefit from separating concerns.

Consider a typical Rails controller, for example a `UsersController` and the `#create` action. By convention, we&#8217;d expect the controller to at the very least know about the model it is tied to (User). But as it often happens, a user being created in the system doesn&#8217;t mean just a `User.create(..)`. It typically implies a series of processing actions that must be completed. Things like sending a confirmation email, generating an invoice, initiating the next step in the workflow or what have you.

Now, when your controller is at the beginning stages of its evolution, these steps can be small and it might not cause you too much pain in your tests to ensure they happen. But as your project grows, you might notice that the amount of setup needed in your tests will increase and your tests will start to assume a rectangular shape. You might also notice that your test has knowledge about (stubs) objects and classes that are not directly related to the responsibility of the object under test. This essentially means that your object under test has dependencies that are getting out of control and is a clear sign that a re-design is needed.

A simple way to get started with removing dependencies is with Plan Old Ruby Objects. Here are some strategies you can use to simplify your controllers:

  1. Identify the steps your controller action is taking that are not to do directly with the model or resource it is tied to.
  2. Create a new Ruby class to encapsulate this behavior and move these steps into this class. Ensure your controller specs still pass after the move.
  3. Copy over the relevant controller specs into the spec file for the new class and modify them to make them pass.
  4. Delete the relevant controller specs. Add a spec to ensure that your new object is called (if you&#8217;re using RSpec, you can mock this with `#expect`)

Now that you&#8217;ve had a brief intro to how tests can signal a design which needs a re-work, next time we&#8217;ll look at how we can use form objects to re-work complex controller actions into something that is easier to reason about.

All the best,

-Sid

P.S. Have any questions about testing or technical debt in Rails? Let me know in a reply to this email and I&#8217;ll follow up as quickly as I can. Looking forward to hearing from you.