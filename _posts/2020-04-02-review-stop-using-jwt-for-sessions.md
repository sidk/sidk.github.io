---
id: 585
title: "Review: Stop Using JWT for Sessions"
date: 2020-04-02T10:18:53-04:00
author: Sid
layout: default
guid: https://ducktypelabs.com/?p=585
permalink: /review-stop-using-jwt-for-sessions/
categories:
  - Security
---
JWTs are super popular. There are numerous articles out there that illustrate various benefits they provide - statelessness, ease of usage, scalability and more.

For the skeptical amongst you, you've probably wondered why JWTs are always promoted without a clear discussion of the tradeoffs, and what their actual downsides are. 

> I don't see the appeal of JWT. Cookie and session based authentication are just so much easier to manage.

> JWTs are always used/recommended/taught but I never see the tradeoffs mentioned. Just one day the community said "this is better". Occasionally you'll see it mentioned that they are stateless and don't need a cache or database lookup. the implications of that, however, are never brought up.

In his article, Sven Slootweg (joepie91) writes elegantly about the [drawbacks of using JWT for sessions](http://cryto.net/~joepie91/blog/2016/06/13/stop-using-jwt-for-sessions/). After defining a few terms, he lists all the claimed advantages of JWTs and one by one, offers up arguments against them. He then moves on to listing the downsides of using JWT as a session mechanism and providing supporting detail for each downside. The article wraps up with a section on what JWTs are good for, and ends urging readers to not use JWTs for persistent, long-lived data.

<img src="/assets/images/lock-and-key.png" alt="jwt lock and key image" style="width: 40%; height: 40%;" align="left"/>

I'll start off this review with a recommendation: [Read Sven's article!](http://cryto.net/~joepie91/blog/2016/06/13/stop-using-jwt-for-sessions/) If you're dubious about the benefits of JWTs, you will be able to articulate to your peers what their disadvantages are after reading it and be more confident in any decision y'all make. Also, regardless of what your current thoughts about JWTs are, you will likely learn a thing or two not only about JWTs, but about how to think about web application security. The article presents several security scenarios that any web developer must grapple with and account for, JWT usage notwithstanding, which makes for some very useful learning.

While Sven covers a lot, there are a few things that could do with some more elaboration and coverage.

## Comparison Semantics

In the section titled &#8220;A note upfront&#8221;, the article makes the distinction between cookies, JWTs and sessions and says that one must compare JWTs to sessions and not to cookies. I don't think JWTs should be compared to either of these. A JWT is a cryptographically signed token and can be used in multiple ways, including as part of a session management system. Moreover, the term &#8220;session&#8221; is pretty vague because implementation details can vary widely (stateless/stateful, storage etc). For the purposes of my review, I'll assume that when Sven said &#8220;The correct comparisons are &#8216;sessions vs JWT'&#8230;&#8221;, he meant &#8220;stateful sessions that don't use JWT&#8221; vs &#8220;stateless sessions that use JWT&#8221;.

## Stateless JWTs

The article mentions two important drawbacks of stateless JWTs, namely:

1. They can't be invalidated without involving some kind of stateful architecture.
2. Data in them can go stale.

While this is true, it is worth noting that _these drawbacks apply to any stateless session mechanism, and are not confined to stateless sessions which use JWTs._ Sessions which are only stored client-side, regardless of if they use JWT, can't be invalidated without some state being persistently maintained.

Rails, for example, defaults to [storing sessions client-side in cookies](https://api.rubyonrails.org/classes/ActionDispatch/Session/CookieStore.html), as an HMAC'ed and encrypted session ID. Though JWTs are not used, these sessions cannot be invalidated. Data staleness is also a potential roadblock here if information in your session has a source of truth that lives server-side and is subject to change.

## JWTs are Better for Cross Domain Authentication?

Cookies (which are used with a typical, &#8220;old-school&#8221; session management system) are usually restricted to one domain. A user authenticated on one domain would therefore still need to authenticate when they are redirected to another domain. To avoid this, we can use JWTs. Both apps would be able to generate and validate signatures (either themselves or through a third party), and the JWT would store all the requisite user information. A stateless session using JWTs, in other words.

Though this seems like a pretty reasonable approach, it has many of the same issues described in Sven's article. The inability to revoke sessions and local storage security are two that immediately come to mind.

To fix this flow and still use JWTs, we can do the following:

1. Continue to use &#8220;old-school&#8221; sessions and cookies on both domains.
2. When issuing a redirect to another domain, add a JWT to the URL as a query parameter, to be used as a _one time token_, with a very short life.
3. Give both domains the ability to validate JWTs in query strings and automatically log a user in and issue a session cookie if a valid JWT is present.

This way, we avoid storing sensitive information in local storage and maintain whatever ability we had to revoke user sessions.

## Storing JWTs in Local Storage

The article is very clear on the ramifications of putting sensitive data in local storage. That being said, it does not address a very common argument (I see it all over various internet forums and comment sections) offered up to lend weight to the practice of storing JWTs in local storage:

> If an attacker manages to inject a malicious script into your front end (which is vulnerable to XSS), then they can use that script to make HTTP requests to your server (directly from the user's browser) and your precious httpOnly cookie (containing the user's valid session ID) will be attached to every request so the server will service them without suspecting anything.

This is a reasonable point, but it misses a security consideration. For the attack described above to work, the user has to be logged in and have the app open in a browser tab, which potentially shortens the exploit window available to the attacker. This will arguably give you more time to respond to an attack. If the app stored JWTs in local storage, an attacker would be able to exfiltrate the token to a place of their choosing. Once they have the token, they could use it stay logged in (obtain refresh tokens etc) and extend the exploit window indefinitely.

Now, you could contend that the seriousness of an XSS vulnerability far surpasses any security gains you might make by storing tokens in httpOnly cookies. Like most security problems though, if you don't have a really good reason to lower your security posture, by however little, I'd argue against it and keep my tokens in cookies.

You will of course have to protect your cookies against CSRF attacks. This, by itself, is seen by some as an argument against putting tokens in cookies. To counter that argument I'd offer the following:

1. XSS attacks are really hard to prevent. Though you should do your best to secure yourself against them, you should in your threat model account for a high likelihood that they will happen anyway.
2. CSRF attacks, although prevalent, are easier to guard against because there are only a few ways a CSRF vulnerability can be exploited.

So, XSS -> high likelihood and CSRF -> low likelihood. I'd rather add anti-CSRF measures (which most frameworks give me for free anyway) than risk my users' tokens (which are equivalent to a username/password combo) being exfiltrated and used indefinitely.

_Note: Sven quotes an article (the &#8220;Storing Sessions&#8221; section) when discussing this. Unfortunately the link to it appears to be dead. Thankfully however, it exists on the wayback machine and can be found [here](https://web.archive.org/web/20150825235139/https://blog.prevoty.com/does-jwt-put-your-web-app-at-risk). I recommend reading it for more insight into the potential flaws of putting JWTs in local storage._

## Issues with the JWT spec

The article concludes by saying that about the only thing JWTs are well suited for are single-use authorization tokens. As an admittedly weak counterpoint to that (because I'm no cryptologist), I'd like to mention that in my reading so far, I have come across multiple cryptographic issues with the the JOSE family of standards (of which the JWT spec is a part). Refer [here](https://news.ycombinator.com/item?id=14292223) for an example. If you're not familiar with `tptacek`, he's a professional security expert, and part of the team which developed the exercises at https://cryptopals.com/. My understanding of the cryptograhic drawbacks of JOSE thus far boil down to the following:

1. It allows for signature generation/validation algorithm choice. This effectively means that as developers, we either have to depend on the JWT library that we're using to implement things securely (there have been several vulnerabilities in libraries reported over the course of the JWT spec's lifespan), or know enough to secure things in the app layer. As a spec, JWT/JOSE could have avoided this by constraining the choices that implementers have and simplifying what the spec is capable of.
2. It allows for a diverse assortment of encryption algorithms to be used, many of which are notoriously hard to get right.

The alternative commonly suggested to a JWT is to use a secure random string instead, or to HMAC, both of which require significantly less code than a spec-compliant JWT library.

Take this section with a grain of salt because I'm not a cryptography expert, but do do your own research into this using the above links as a starting point.

---

## What's next?

Now that you've read Sven's article and reached the end of my review, what are next actions you can take?

If you're "stuck" with using JWTs or still believe in their usefulness for maintaining sessions, realize that the arguments raised in this article against JWTs can still be helpful. Regardless of which side you're on, I think everyone agrees with the fact that users and their information must be kept safe.

If you haven't already, check out my article on [mistakes that can be made when using JWTs](https://www.ducktypelabs.com/5-mistakes-web-developers-should-avoid-when-using-jwts-for-authentication/).

If you were skeptical of JWTs to begin with and were having a hard time articulating what their disadvantages are, I hope you now have enough material to bring up in discussion with your peers 😁. [Write me](mailto://sidk@ducktypelabs.com) and let me know how it goes - I'd love to hear from you!

I'm currently learning and writing all I can about JWTs and Authentication. Subscribe below to get notified whenever I publish a new article.
