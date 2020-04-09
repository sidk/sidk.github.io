---
id: 378
title: My testing workflow for a Rails app
date: 2016-05-29T22:13:58-04:00
author: Sid
layout: revision
guid: http://ducktypelabs.com/368-revision-v1/
permalink: /368-revision-v1/
---
When I started working with Ruby and Rails a few years ago, I was introduced to the concept of TDD. Michael Hartl&#8217;s tutorial, which most recent Rails developers start with, has TDD baked in. In theory, TDD sounded great but as I found out, practicing it was another story.

> Am I testing too much? I feel like I&#8217;m testing not enough and there&#8217;s use cases which will fall through the gaps of testing. Serious question though, I&#8217;ve spent more time testing than writing functionality.
> 
> I&#8217;m stuck on TDDing because I&#8217;m unsure what my object&#8217;s interface should look like.
> 
> Am I a bad developer if I don&#8217;t TDD?

These are quotes from real Rails developers, and I&#8217;ve certainly identified with these concerns at one point or another. If you&#8217;re here, I hope to convince you that it is more than possible to have well tested code and a development workflow that you enjoy.

These are all concerns that went through my head too at some point as I progressed. Over time, I&#8217;ve grown to adopt a testing workflow that has served me well in churning out production code. Hopefully you&#8217;ll find it useful to explore what it is, and will make you consider your own testing workflow in depth.

Note: Because I primarily use RSpec, I will use the terms &#8220;test&#8221; and &#8220;spec&#8221; interchangeably.

## Why TDD

First, we must make the distinction between TDD and automated tests in general.

Tests, in general, can be derived from TDD or written after-the-fact. Either way, a test suite which covers the majority of code paths within your app has a couple of huge benefits:

  1. You can change code with a high degree of confidence that you won&#8217;t introduce bugs. Good, because change is constant.
  2. It documents your code. Important whether you work in a team or not.

TDD, as you probably know, is a workflow where tests are written first and drive codebase development & design. The main benefit of TDD is that it enables a tight feedback loop when you&#8217;re developing (the red-green-refactor cycle). In addition to being enjoyable, this helps to keep the cost of changing your code constant.

For a good analysis on the benefits read this: http://scrumology.com/the-benefits-of-tdd-part-2/

I&#8217;m not going to say TDD is a must. However, I&#8217;d recommend it because:

  1. It pushes me to achieve a higher level of test coverage than I would&#8217;ve had otherwise. This is important because on the whole, a higher test coverage means a lower incidence of bugs.
  2. I enjoy the process once I&#8217;m in the groove. 

## Where TDD Fits In

That being said, I don&#8217;t always TDD:

  1. If I&#8217;m doing something totally new or something I&#8217;m unsure how to implement, then I first develop working code without tests. Then I throw this away, and restart development with TDD. This way I know what has to be done, but can work out my design with tests leading the way.
  2. If I&#8217;m changing the behavior of an existing feature, then I TDD right away (assuming the code has tests to begin with) 
  3. If the code doesn&#8217;t have tests, then I introduce test coverage before changing the code. This can be a chicken & egg problem sometimes, because you might need to change the code to introduce tests. 
  4. When I&#8217;m fixing a bug, I write a test to prove the failure before I go in and fix it.

How Sarah Mei tests: http://www.sarahmei.com/blog/2010/05/29/outside-in-bdd/

http://martinfowler.com/bliki/UnitTest.html

<div id="fb-root">
</div>



<div class="fb-post" data-href="https://www.facebook.com/notes/kent-beck/rip-tdd/750840194948847/" data-width="698">
  <blockquote cite="https://www.facebook.com/notes/kent-beck/rip-tdd/750840194948847/" class="fb-xfbml-parse-ignore">
    <p>
      Posted by <a href="https://www.facebook.com/kentlbeck">Kent Beck</a> on&nbsp;<a href="https://www.facebook.com/notes/kent-beck/rip-tdd/750840194948847/">Tuesday, April 29, 2014</a>
    </p>
  </blockquote>
</div>

## What should I test in my Rails app?

The rest of this article will attempt to answer the question: &#8220;What should I test in my Rails app?&#8221;.

### Install & Setup `simplecov`

You need a way to assess test coverage in your system. If you don&#8217;t have one already, I recommend installing the [`simplecov`](https://github.com/colszowka/simplecov) gem. I&#8217;d also recommend running a coverage tool as part of your CI process.

### Set up a key command to quickly run the test you&#8217;re working on

You will likely find it beneficial as you&#8217;re writing tests to get immediate feedback. A key command like `\q` or `Enter` works beautifully for this purpose.

### Acceptance Tests

Tests involving the browser are typically slow. At the same time, without acceptance tests, you can&#8217;t be 100% sure that all the pieces of your system work as expected. What I do is this:

  * Write an acceptance test for the &#8220;happy path&#8221; (elaborate)
  * Save testing the edge cases for the controller (or other objects) tests
  * Rely on CSS class & id visibility for your test assertions. This ensures your tests don&#8217;t have to change too often. For example, favor something like `expect(page).to have_css('#greeting')` over `expect(page).to have_content('hello')` (double check code)
  * [Use a headless browser](https://github.com/teampoltergeist/poltergeist)

### Controllers

I prefer to keep my controller specs small and integrated (for the most part). With my controller tests, I get two extra benefits:

  1. The tests won&#8217;t pass if the route is not defined.
  2. By using [`render_views`](https://www.relishapp.com/rspec/rspec-rails/docs/controller-specs/render-views), I ensure that any view errors are caught.

**Because of this, I don&#8217;t write view and route tests.**

At a minimum, here&#8217;s what I test for in my controllers:

  * Check that the right view template is rendered
  * Check redirects
  * Ensure flash messages are set
  * Ensure instance variables are set as expected
  * If applicable (and simple enough to do so), check that emails are sent and/or the database is updated

Ideally, I try to stay clear of putting business logic in my controllers. If my controller spec has tests for more than the above, I start to consider extraction.

### Models

In my model tests, I often make use of the [`build_stubbed`](https://robots.thoughtbot.com/use-factory-girls-build-stubbed-for-a-faster-test) helper that FactoryGirl provides. It reduces your test&#8217;s dependence on the database and makes for a faster test.

My model specs typically only contain tests for the following:

  * Columns (something like `it { is_expected.to have_db_column(:name).of_type(:string) }`)
  * Scopes
  * Validations
  * Associations
  * Delegations

If your spec contains tests for more than the above, you might want to consider if extracting this functionality might make your code easier to deal with.

### POROs and Other Custom Rails Objects

Most business logic in a decent sized Rails app will (or should) reside outside the MVC structure. Subsequently, the lion&#8217;s share of your tests will run on these objects:

  * Validators
  * Serializers
  * Workers 
  * Helpers 
  * Mailers
  * POROs (things like (Form Objects)[http://ducktypelabs.com/how-to-keep-your-controllers-thin-with-form-objects/], Value Objects etc)

As far as you can, aim to isolate your tests for these objects from other parts of the system. Though it won&#8217;t always be possible because of the way Rails is designed, sustained effort in this area is bound to pay off in more ways than one:

  1. Your tests will run faster. This will make your development process more enjoyable.
  2. Difficulty isolating an object under test (for example, with deeply nested stubs) can be a sign that your object is doing too much, or knows too much about other objects in the system. When this happens to me, I often reconsider my design and think of ways I can reduce my object&#8217;s responsibilities.

### Test Javascript

I use (jasmine)[http://jasmine.github.io/] to write my Javascript specs. I do think Javascript specs are a necessary component of a well-tested and stable Rails app. There&#8217;s not much more to say here other than the fact that Javascript code is like any other code &#8211; it is subject to change over time. The higher your test coverage, the more confidence you will have in changing your code when the time comes to do so.

### External Services

Calling external services directly in your specs can cause multiple issues:

  * Slower tests
  * Intermittent test failures
  * Hitting API rate limits on 3rd party sites (e.g. Twitter).
  * Service may not exist yet (only documentation for it).
  * Service doesn’t have a sandbox or staging server.

### Thoughts on test speed

Test speed is also important. As Martin Fowler says:

> &#8230;your test suites should run fast enough that you&#8217;re not discouraged from running them frequently enough. And frequently enough is so that when they detect a bug there&#8217;s **a sufficiently small amount of work to look through that you can find it quickly.**(emphasis mine)

Kent Beck&#8217;s rule of thumb: Your test suite should take no longer than 10 minutes to run.

## Conclusion/Action Items/CTA