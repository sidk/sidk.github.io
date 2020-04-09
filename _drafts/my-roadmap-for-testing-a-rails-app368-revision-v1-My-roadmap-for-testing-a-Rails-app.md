---
id: 420
title: My roadmap for testing a Rails app
date: 2016-07-22T11:46:00-04:00
author: Sid
layout: revision
guid: http://ducktypelabs.com/368-revision-v1/
permalink: /368-revision-v1/
---
TDD, and testing in general, is a ubiquitous concept in the Ruby and Rails world. The popular tutorials (like Michael Hartl&#8217;s tutorial) have TDD baked in, and newcomers are encouraged to adopt the practice. However, **testing Rails apps can be challenging.**

You might feel like you&#8217;re testing too much, or that you&#8217;re not testing enough. You might feel like you&#8217;re spending more time testing than writing functionality. You might be stuck on TDDing because you&#8217;re unsure what your object&#8217;s interface should look like. **Worst case, you might feel like you&#8217;re a bad developer if you don&#8217;t TDD.**

Fear not, because it is more than possible to have well tested code and an enjoyable workflow that leaves you feeling confident with your code. In this article, I&#8217;ll show you a roadmap that I use regularly to churn out production Rails apps. The roadmap will give you an overview of what parts of your Rails app ought to be covered with tests. Along the way, I&#8217;ll share the techniques I use in my tests to ensure a good level of coverage, and at the same time feel the joy of programming as often as possible.

_Note: I use the terms &#8220;test&#8221; and &#8220;spec&#8221; interchangeably._  
_Disclaimer: Depending on your needs, your mileage may vary with the following suggestions. I hope you learn something either way._

## Why TDD

First, we must make the distinction between TDD and automated tests in general.

Tests, in general, can be derived from TDD or written after-the-fact. Either way, a test suite which covers the majority of code paths within your app has a couple of huge benefits:

  1. You can change code with a high degree of confidence that you won&#8217;t introduce bugs.
  2. It documents your code.

TDD, as you probably know, is a workflow where tests are written first and drive codebase development & design. The main benefit of TDD is that it enables a tight feedback loop when you&#8217;re developing (the red-green-refactor cycle). In addition to being enjoyable, this helps to keep the cost of changing your code constant.

Though TDD isn&#8217;t a must, I find it useful because:

  1. It pushes me to achieve a higher level of test coverage than I would&#8217;ve had otherwise. This is important because on the whole, a higher test coverage means a lower incidence of bugs.
  2. I enjoy the process once I&#8217;m in the groove. 

## Where TDD Fits In

That being said, I don&#8217;t always TDD:

  1. If I&#8217;m unsure how to implement something, I first develop working code without tests. Then I throw this away, and restart development with TDD. This way I know what has to be done, but can work out my design with tests leading the way.
  2. If I&#8217;m changing the behavior of an existing feature, then I TDD right away (assuming the code has tests to begin with) 
  3. If the code doesn&#8217;t have tests, then I introduce test coverage before changing the code. This can be a chicken & egg problem sometimes, because you might need to change the code to introduce tests. 
  4. When I&#8217;m fixing a bug, I write a test to prove the failure before I go in and fix it.

## The Rails Testing Roadmap

#### **Install & Setup `simplecov`**

You need a way to assess test coverage in your system. If you don&#8217;t have one already, I recommend installing the [`simplecov`](https://github.com/colszowka/simplecov) gem. I&#8217;d also recommend running a coverage tool as part of your CI process.

#### **Set up a key command to quickly run the test you&#8217;re working on**

You will likely find it beneficial as you&#8217;re writing tests to get immediate feedback. A key command like `\q` or `Enter` works nicely for this purpose.

#### **Acceptance Tests**

Tests involving the browser are typically slow. At the same time, without acceptance tests, you can&#8217;t be 100% sure that all the pieces of your system work as expected. So a happy medium is this:

  * Write an acceptance test for the &#8220;happy path&#8221;
  * Save testing the edge cases for the controller (or other objects) tests
  * Rely on CSS classes & ids for your test assertions. This ensures your tests don&#8217;t have to change too often. For example, favor something like `expect(page).to have_css('#greeting')` over `expect(page).to have_content('hello')`
  * [Use a headless browser](https://github.com/teampoltergeist/poltergeist)

#### **Controllers**

I prefer to keep my controller specs small and integrated (for the most part).

At a minimum, here&#8217;s what I test for in my controllers:

  * Check that the right view template is rendered
  * Check redirects
  * Ensure flash messages are set
  * Ensure instance variables are set as expected
  * If applicable (and simple enough to do so), check that emails are sent and/or the database is updated

Ideally, I try to stay clear of putting business logic in my controllers. **If my controller spec has tests for more than the above, I start to consider extracting the relevant functionality into service objects.**

I also get two extra benefits with my controller tests:

  1. The tests won&#8217;t pass if the route is not defined.
  2. By using [`render_views`](https://www.relishapp.com/rspec/rspec-rails/docs/controller-specs/render-views), I ensure that any view errors are caught.

**Because of this, I don&#8217;t write view and route tests.**

#### **Models**

In my model tests, I often make use of the [`build_stubbed`](https://robots.thoughtbot.com/use-factory-girls-build-stubbed-for-a-faster-test) helper that FactoryGirl provides. It reduces your test&#8217;s dependence on the database and makes for a faster test.

My model specs typically only contain tests for the following:

  * Columns (something like `it { is_expected.to have_db_column(:name).of_type(:string) }`)
  * Scopes
  * Validations
  * Associations
  * Delegations

If your spec contains tests for more than the above, you might want to consider if extracting this functionality would make your code easier to deal with.

#### **POROs and Other Custom Rails Objects**

Most business logic in a decent sized Rails app will (or should) reside outside the MVC structure. Subsequently, the lion&#8217;s share of your tests will run on these objects:

  * Validators
  * Serializers
  * Workers 
  * Helpers 
  * Mailers
  * POROs (things like [Form Objects](http://ducktypelabs.com/how-to-keep-your-controllers-thin-with-form-objects/), Value Objects etc)

As far as you can, aim to isolate your tests for these objects from other parts of the system. Though it won&#8217;t always be possible because of the way Rails is designed, sustained effort in this area is bound to pay off in more ways than one:

  1. Your tests will run faster. This will make your development process more enjoyable.
  2. Difficulty isolating an object under test (for example, with deeply nested stubs) can be a sign that your object is doing too much, or knows too much about other objects in the system. When this happens to me, I often reconsider my design and think of ways I can reduce my object&#8217;s responsibilities.

#### **Javascript**

I use [jasmine](http://jasmine.github.io/) to write my Javascript specs. I do think Javascript specs are a necessary component of a well-tested and stable Rails app. There&#8217;s not much more to say here other than the fact that Javascript code is like any other code &#8211; it is subject to change over time. **The higher your test coverage, the more confidence you will have in changing your code when the time comes to do so.**

#### **External Services**

You can run into a number of issues when dealing with external services in your tests:

  * Slow tests, because you need to wait for the 3rd party API call to return.
  * Intermittent test failures.
  * Hitting API rate limits.
  * Service doesn’t have a &#8220;test&#8221; mode.

To get around these issues, I use the [`VCR`](https://github.com/vcr/vcr) and [`WebMock`](https://github.com/vcr/vcr) gems. These gems allow you to effectively stub the calls made to third party services, and ensure that your tests are deterministic and fast. **A word of caution:** If you decide to use gems like VCR and WebMock, you should always keep an eye out for changes to the API. When your calls to the API are stubbed and the API changes, your tests can continue to pass even though your production app might break.

#### **Test Speed: Rules of Thumb**

Martin Fowler says:

> &#8230;your test suites should run fast enough that you&#8217;re not discouraged from running them frequently enough. And frequently enough is so that when they detect a bug there&#8217;s **a sufficiently small amount of work to look through that you can find it quickly.** _(emphasis mine)_

[Kent Beck&#8217;s rule of thumb: Your test suite as a whole should take no longer than 10 minutes to run.](http://martinfowler.com/bliki/UnitTest.html)

## Conclusion

How do you test your Rails apps? Has TDD helped or hindered you? Compare your own test strategy to the one described above and consider how they differ, and if either can be improved. More importantly, figure out what your goals are with your tests, and evaluate if they are giving you the results you want.