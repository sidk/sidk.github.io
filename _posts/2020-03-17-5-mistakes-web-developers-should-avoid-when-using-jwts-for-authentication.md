---
id: 564
title: Are you making these JWT Authentication mistakes?
date: 2020-03-17T13:46:56-04:00
author: Sid
layout: post
permalink: /5-mistakes-web-developers-should-avoid-when-using-jwts-for-authentication/
categories:
  - Security
---

JWTs have received and continue to receive a lot of positive and negative attention over the last few years. The pro-JWT camp touts benefits like statelessness, portability, a convenient interface and so on. The anti-JWT camp says JWT is a kitchen sink of crypto formats and maximizes the number of things that can go wrong.

It's hard to figure out what "best practices" are in such an environment. When you're tasked with building a web app authentication system and have to make decisions on what techs you'll use and how you'll put them together with tech you're stuck with or want to explore, coming across conflicting advice can be unsettling. 

Especially since any slip-ups could end up hurting the very users you're trying to protect. Are you doomed if you don't follow every piece of advice out there?

`¯\_(ツ)_/¯`

In this post, I&#8217;ll take a pragmatic approach and draw up a short list of mistakes that can be made when using JWTs for authentication, while trying my best not to make any value judgements. Think of this as a checklist to go through as you&#8217;re building authentication. Staying clear of these mistakes, or having well thought-out reasons for not doing so, will go a long way in helping you build a more secure authentication system.

To keep things short, the scenarios I&#8217;m going to focus on all involve a front-end (run on a browser), user-facing application communicating with one or more servers to perform authentication.

## Not having a strategy for session invalidation or revocation

In this scenario, you have a front-end application which makes authentication requests to a server by sending it a username and password. The server verifies this username/password combination, generates a JWT and sends it back to the client. The client is then able to make further requests using the JWT, and the server verifies these requests by validating the signature on the JWT, and saves a database call that it would have otherwise had to make.

There is a problem with this approach, however. A JWT is valid as long as:

  1. It hasn&#8217;t expired yet, and 
  2. It has a valid cryptographic signature

If you&#8217;re ever in a situation where you want to reject requests made with a valid JWT, you&#8217;ll need to have a strategy and corresponding infrastructure/app behaviour in place. A common scenario often cited is when a malicious account with a valid JWT needs to be blocked from making any further requests to your servers.

There are a few options at your disposal to address this particular situation if you think it&#8217;s something to be addressed. Having a low expiry period and a longer lived &#8220;refresh&#8221; token, maintaining a &#8220;deny list&#8221; in some external store, changing the signing key, and potentially others. Every option comes with its own set of trade-offs, so do take the time to research them and understand what you&#8217;re giving up and what you&#8217;re gaining.

## Putting sensitive data in the JWT

A typical JWT looks like this:

    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
    
Though this looks illegible, it is actually very readable because it&#8217;s a plain text JSON object [encoded](https://en.wikipedia.org/wiki/Binary-to-text_encoding) using base64. For every piece of information you decide to put in your JWT, ask yourself: How much damage would it cause if this gets into the wrong hands?

Two common remediations for having sensitive data in the JWT are:

  1. Stop putting sensitive data in there.
  2. If you really need to have sensitive data, encrypt your JWT.

Needless to say, there are trade-offs with both approaches, which neccessitates you having a good think about what your requirements are. For example, with option 1, you might have to change your application architecture and perhaps incur a lookup cost. Option 2 on the other hand, adds the complexity of having to encrypt something and thinking about where and how it will be decrypted.

## Not guarding against CSRF (cross-site request forgery)

In this scenario, your front-end app stores the JWT in a cookie (with no protections on it) and your server authenticates requests by verifying it. If a user has your app open in a browser tab and happens upon a malicious web site in another tab, this web site can make authenticated requests to your web app because the browser will automatically include any cookies associated with your web app&#8217;s domain. This is known as cross-site request forgery, and you&#8217;ll want to [guard against it](https://en.wikipedia.org/wiki/Cross-site_request_forgery#Prevention). A common solution is for your server to include an anti-CSRF token in its responses and require that any requests made to it also contain a matching token, the main idea being that a malicious script would have no way of knowing what this token is and therefore have its requests blocked.

If you&#8217;re using a framework like Rails or Express, the chances are high that the framework will include options that you can use to protect your users against CSRF, using one or more of the ways detailed in the Wikipedia link above.

A measure you can take in parallel to implementing an anti-CSRF token system is to use the `SameSite` flag on your cookies. Setting `SameSite=Strict` will tell the browser that it should only send cookies for requests originating from the site that set the cookie.

_Related:_ You&#8217;ll also want to set the `httpOnly` and `secure` flags on your cookies. `httpOnly` prevents malicious scripts from accessing and potentially exfiltrating the contents of your JWT (in the context of an [XSS](https://en.wikipedia.org/wiki/Cross-site_scripting) attack) and `secure` ensures that cookies will not be sent across the wire unless the [https](https://en.wikipedia.org/wiki/HTTPS) protocol is being used.

## Storing the JWT in local storage

Since cookies typically have a 4kb storage limit, you might reason that a JWT that contains more information could be stored in local storage instead.

### Local storage is not protected against XSS

If you have an XSS vulnerability, a malicious script could easily exfiltrate the contents of local storage to anywhere it chooses to, allowing the attacker to impersonate the user at their convenience. A common counterpoint to this argument is that in the event of an XSS attack against an app that stores its JWT in a cookie, an attacker would still be able to make authenticated requests since `httpOnly` and `secure` do nothing to prevent this. This is true, but it is arguably more secure because it limits the attacker to carrying out specific attacks only when the user is actively using the app, or has it open in a tab somewhere.

I&#8217;d bias towards choosing the more secure option in this case (and also do my best not to introduce XSS vulnerabilities!).

## Choosing an asymmetric encryption algorithm without a good reason

I&#8217;m not a security expert, so take this point with a grain of salt. From the research I&#8217;ve done so far, RSA (the most commonly used crypto algorithm to asymmetrically sign JWTs) is hard to get right and can have subtle implementation bugs. Cryptographers with way more knowledge than me have recommended [against](https://latacora.singles/2018/04/03/cryptographic-right-answers.html) [it](https://latacora.micro.blog/2019/07/24/how-not-to.html)

Unless you have a really good reason for choosing asymmetric/public key encryption, choose HS256 (HMAC-SHA256) as your algorithm. If you&#8217;re building a system where a third party needs to be able to validate a signature but not be able to generate one (what folks typically use asymmetric encryption in a web app for), consider an API where the third party can ask the secret holder if a given signature is valid.

Also don&#8217;t allow &#8220;None&#8221; as one of the algorithm choices 🙂

To learn more about how these algorithms work and how they can be exploited, I highly recommend [the cryptopals challenges](https://cryptopals.com/sets/6)!


## What's next?

Now that you know of a few things that can go wrong when using JWTs to build authentication, here's what you can do to solidify your knowledge.

Pick a codebase to "audit" (something you're hopefully pretty familiar with). Identify places where you may have made one or more of the mistakes above. Ask yourself:

* If there was a good justification for it
* If there wasn't, how you would fix it


If you're using JWTs for session management, you'll likely find [my article reviewing the drawbacks of this approach useful]({% post_url 2020-04-02-review-stop-using-jwt-for-sessions %}).


* * *

Finally, if you found this article useful, drop your email in the box below to stay updated when I publish new articles. I'm currently learning all I can about authentication systems and am aiming to publish a new article once every two weeks.
