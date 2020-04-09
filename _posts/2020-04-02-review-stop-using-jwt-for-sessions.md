---
id: 585
title: 'Review: Stop Using JWT for Sessions'
date: 2020-04-02T10:18:53-04:00
author: Sid
layout: post
guid: https://ducktypelabs.com/?p=585
permalink: /review-stop-using-jwt-for-sessions/
classic-editor-remember:
  - classic-editor
cta_content_placement:
  - below
categories:
  - Security
---
If you&#8217;ve read any article or tutorial on JWTs, you&#8217;ve probably heard a lot about how great they are. They&#8217;re stateless, self-contained, highly scalable, easy to work with and more. You&#8217;ve probably also wondered why JWTs are always used/recommended/taught without a clear discussion of the tradeoffs, and what their actual downsides are.

In his article, Sven Slootweg (joepie91) writes elegantly about the [drawbacks of using JWT for sessions](http://cryto.net/~joepie91/blog/2016/06/13/stop-using-jwt-for-sessions/). After defining a few terms, he lists all the claimed advantages of JWTs and one by one, offers up arguments against them. He then moves on to listing the downsides of using JWT as a session mechanism and providing supporting detail for each downside. The article wraps up with a section on what JWTs are good for, and ends urging readers to not use JWTs for persistent, long-lived data.

I&#8217;ll start off this review with a recommendation: [Read Sven&#8217;s article!](http://cryto.net/~joepie91/blog/2016/06/13/stop-using-jwt-for-sessions/) If you&#8217;re dubious about the benefits of JWTs, you will be able to articulate to your peers what their disadvantages are after reading it and be more confident in any decision you or y&#8217;all make. Also, regardless of what your current thoughts about JWTs are, you will likely learn a thing or two not only about JWTs, but about how to think about web application security. The article presents several security scenarios that any web developer must grapple with and account for, JWT usage notwithstanding, which makes for some very useful learning.

While Sven covers a lot, there are a few things that could do with some more elaboration and coverage.

## Comparison Semantics

In the section titled &#8220;A note upfront&#8221;, the article makes the distinction between cookies, JWTs and sessions and says that one must compare JWTs to sessions and not to cookies. I don&#8217;t think JWTs should be compared to either of these. A JWT is a cryptographically signed token and can be used in multiple ways, including as part of a session management system. Moreover, the term &#8220;session&#8221; is pretty vague because implementation details can vary widely (stateless/stateful, storage etc). For the purposes of my review, I&#8217;ll assume that when Sven said &#8220;The correct comparisons are &#8216;sessions vs JWT&#8217;&#8230;&#8221;, he meant &#8220;stateful sessions that don&#8217;t use JWT&#8221; vs &#8220;stateless sessions that use JWT&#8221;.

## Stateless JWTs

The article mentions two important drawbacks of stateless JWTs, namely:

  1. They can&#8217;t be invalidated without involving some kind of stateful architecture.
  2. Data in them can go stale.

While this is true, it is worth noting that _these drawbacks apply to any stateless session mechanism, and are not confined to stateless sessions which use JWTs._ Sessions which are only stored client-side, regardless of if they use JWT, can&#8217;t be invalidated without some state being persistently maintained.

Rails, for example, defaults to [storing sessions client-side in cookies](https://api.rubyonrails.org/classes/ActionDispatch/Session/CookieStore.html), as an HMAC&#8217;ed and encrypted session ID. Though JWTs are not used, these sessions cannot be invalidated. Data staleness is also a potential roadblock here if information in your session has a source of truth that lives server-side and is subject to change.

## JWTs are Better for Cross Domain Authentication?

A commonly touted advantage of JWT is the fact that they are seemingly well suited for use across different domains. If you&#8217;re wondering why this would be useful, think of a banking app that wants to redirect you to a retirement planning app without forcing you to log in a second time (assuming the same vendor provides both services, but has developed two separate apps deployed on separate domains). The sequence of ideas that lead to this conclusion can be plotted out as follows:

  1. Cookies (which are used with a typical, &#8220;old-school&#8221; session management system) are usually restricted to one domain.
  2. Because we can&#8217;t share cookies, this means that a user authenticated on one domain will still need to go through an authentication flow when they are redirected to another domain.
  3. To avoid this, we can use JWTs. Both apps would be able to generate and validate signatures (either themselves or through a third party). The JWT would store all the requisite user information and we would pass it around by storing it in local storage. A stateless session using JWTs, in other words.

Though this seems like a pretty reasonable approach, it has many of the same issues described in Sven&#8217;s article. The inability to revoke sessions and local storage security are two that immediately come to mind.

To fix this flow and still use JWTs, we can do the following:

  1. Continue to use &#8220;old-school&#8221; sessions and cookies on both domains.
  2. When issuing a redirect to another domain, add a JWT to the URL as a query parameter, to be used as a _one time token_, with a very short life.
  3. Give both domains the ability to validate JWTs in query strings and automatically log a user in and issue a session cookie if a valid JWT is present.

This way, we avoid storing sensitive information in local storage and maintain whatever ability we had to revoke user sessions.

## Storing JWTs in Local Storage

The article is very clear on the ramifications of putting sensitive data in local storage. That being said, it does not address a very common argument (I see it all over various internet forums and comment sections) offered up to lend weight to the practice of storing JWTs in local storage:

> If an attacker manages to inject a malicious script into your front end, then they can use that script to make HTTP requests to your server (directly from the user&#8217;s browser) and your precious httpOnly cookie (containing the user&#8217;s valid session ID) will be attached to every request so the server will service them without suspecting anything.

This is a reasonable point, but it misses a security consideration. For the attack described above to work, the user has to be logged in and have the app open in a browser tab, which potentially shortens the exploit window available to the attacker. This will arguably give you more time to respond to an attack. If the app stored JWTs in local storage, an attacker would be able to exfiltrate the token to a place of their choosing. Once they have the token, they could use it stay logged in (obtain refresh tokens etc) and extend the exploit window indefinitely.

Admittedly, this is a fine point. You could also contend that the seriousness of an XSS vulnerability far surpasses any security gains you might make by storing tokens in httpOnly cookies. Like most security problems though, if you don&#8217;t have a really good reason to lower your security posture, by however little, I&#8217;d argue against it and keep my tokens in cookies.

_Note: Sven quotes an article (the &#8220;Storing Sessions&#8221; section) when discussing this. Unfortunately the link to it appears to be dead. Thankfully however, it exists on the wayback machine and can be found [here](https://web.archive.org/web/20150825235139/https://blog.prevoty.com/does-jwt-put-your-web-app-at-risk). I recommend reading it for more insight into the potential flaws of putting JWTs in local storage._

## Summary and Conclusions

The article concludes by saying that about the only thing JWTs are well suited for are single-use authorization tokens. As an admittedly weak counterpoint to that (because I&#8217;m no cryptologist), I&#8217;d like to mention that in my reading so far, I have come across multiple cryptographic issues with the the JOSE family of standards (of which the JWT spec is a part). Refer [here](https://news.ycombinator.com/item?id=14292223) for an example. If you&#8217;re not familiar with `tptacek`, he&#8217;s a professional security expert, part of the team which developed the exercises at https://cryptopals.com/ and says several smart things. My understanding of the cryptograhic drawbacks of JOSE thus far boil down to the following:

  1. It allows for signature generation/validation algorithm choice. This effectively means that as developers, we either have to depend on the JWT library that we&#8217;re using to implement things securely (there have been several vulnerabilities in libraries reported over the course of the JWT spec&#8217;s lifespan), or know enough to secure things in the app layer. As a spec, JWT/JOSE could have avoided this by constraining the choices that implementers have and simplifying what the spec is capable of.
  2. It allows for a diverse assortment of encryption algorithms to be used, many of which are notoriously hard to get right.

* * *

To wrap up, I&#8217;d recommend that if you want to use JWTs as an authentication or authorization solution, to acquire as much knowledge as you feasibly can to feel confident in your decision and implementation. If you haven&#8217;t already, check out my previous article on [mistakes that can be made when using JWTs](https://ducktypelabs.com/5-mistakes-web-developers-should-avoid-when-using-jwts-for-authentication/).

Hope you enjoyed the read &#8211; please let me know your thoughts in the comments below. If you liked what you read, make sure to drop your email in the form below to get updated whenever I publish a new article!

<div id="mc_embed_signup" style="background: #dff7fe; padding: 15px;">
</div>