---
title: 'Is putting JWTs in local storage "bad"?'
date: 2020-04-29T10:18:53-04:00
author: Sid
layout: default
permalink: /is-localstorage-bad/
categories:
  - Security
---

You might've heard that you shouldn't store JWTs (or any sensitive data for that matter) in [local storage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage) because if your app was vulnerable to XSS, an attacker would easily be able to access the token and impersonate the user. 

You've probably _also_ heard that it is, in fact, alright to store JWTs in local storage because if your app was vulnerable to XSS you'd have much bigger problems to worry about than an attacker having unfettered access to an auth token. You might've even heard that keeping JWTs in local storage eliminates the possibility that your users will be attacked with CSRF. 

If you've heard both points of view and are still feeling stuck on how best to think about the security of your own system and actually build something, read on. This article will hopefully help.

Like any good debate (TODO: link to comment threads, one per word), both sides have valid points. The tricky, and _important_, thing for you, a web app developer, is to figure out two things:
1. What the trade-offs are with each choice.
2. Your comfort level with the worst case scenarios each choice implies

Once you've worked this out, you'll feel more confident in the eventual choice you make. If your comfort level is roughly the same for both choices, you can pick the one which is easier to do. Alternatively, if one of the choices makes it easier for you to sleep at night, you might decide that it's worth the extra effort.

Note: Although I'm focusing on JWTs, the points in this discussion apply to any sensitive information that the client needs to store

## [XSS](https://owasp.org/www-community/attacks/xss/) is bad

You should be doing all you can to protect yourself against XSS. Regardless of where you put your JWT.

However, when you're thinking about the security of your system, I believe *you should treat XSS as an inevitability.* Especially if your app depends on third party scripts from sources you don't control. 

## But does having an XSS vulnerability mean it's "game over"?

[CHESS ILLUSTRATION HERE?]

If we were playing a game of chess, and I captured your queen, would you abandon hope and throw in the towel? I don't think so; you'd probably just regroup and continue to try your best to win the game. If you were an experienced chess player, you might even have a set of go-to tactics to bring into play when you lose your queen.

The strategy for dealing with an XSS vulnerability is very similar. Instead of assuming all is lost, you'll want to try your best to chart out one or more courses an attacker could take when exploiting the vulnerability in your app, and put in place defences that minimize the amount of subsequent damage.

This way of thinking is called [defence in depth](https://en.wikipedia.org/wiki/Defense_in_depth_(computing)).

## "Traditional" web apps 

When you're building an app where the following are true:
* A server which you control does authentication
* The JWT does not contain any information that you need to read with JS on the front-end, and is used for authentication
* Your users' sessions could be stateless or stateful

Putting your JWT in an `httpOnly` + `secure` + `sameSite=strict` cookie is *more secure* than putting it in local storage. Why?

* It is not accessible to javascript. This is good, because it means that the token [can't be stolen](https://dev.to/oktadev/what-happens-if-your-jwt-is-stolen-298d)
* Protections against [CSRF (a vulnerability you open yourself up when you store the JWT in a cookie)](https://owasp.org/www-community/attacks/csrf) are easy to add. In addition to the `sameSite` flag that you'd set on your cookies, you could also set up anti-CSRF protections on the server (most server frameworks have solutions for this)
* Reduces amount of time attackers have to actually exploit the vulnerability
* You'd write slightly less code on the front-end because you can take advantage of browser behavior (submits cookie with requests, server can set expiry, lower risk of sending cookie to server unencrypted and potentially other things) 


## Is the increase in security meaningful to you?

Perhaps you have constraints. Maybe the server you're talking to doesn't use the `Set-Cookie` header and you have no option but to store the token somewhere accessible to javascript. Maybe the JWT contains some information that your front-end needs to read. Does this mean you're done for?

"Done for" means different things in different circumstances. Also _how_ "done for" you are matters a lot. 

If you're a bank, you probably don't want to leave any security holes open, no matter how small. The fear of users having their tokens stolen and losing money probably keeps you up at night and you'd feel measurably better if you could prevent attackers from going an extra step further. 

Here's a hypothetical scenario in which storing the JWT in a (`secure` + `httpOnly` + `sameSite`)cookie is arguably better:

* Day 1, attacker discovers XSS vulnerability in bank sofware and deploys exploit. The idea is to transfer money to the attacker's account in small chunks, so as not to raise any suspicions.
* Day 4, bank software team discovers and patches the vulnerability.

If the JWT is stored in a cookie, the attacker has 4 days to run their exploit, and is limited to running it when users are logged in. If the JWT is stored in localStorage, the attacker has potentially until the bank discovers the theft (which theoretically could be a long time), assuming that they can exfiltrate a longer-lived refresh token as well. 

Of course, as a bank, you would not rely on lack of javascript access alone to increase your security. You would probably put as many defences in place as you can think of to reduce the amount of time between the attacker's deployment of the exploit and the time you discover that it's happening.

On the other hand, if you're someone like reddit, maybe the increased risk of users having their tokens stolen doesn't meaningfully change your answer to the question "How screwed are we if we're vulnerable to XSS?". You may figure that doing your best to protect yourself against XSS and minimizing token lifetimes is good enough and put your JWT in local or session storage because it gives you other benefits. This is OK too!

The important thing is to take the time to think through these questions.

## Talk to your team! 

Not only the devs, but everyone. Security is an organizational concern and the insights you get from talking to folks who are not developers will be highly informative. They'll help you reason about the threats you face and their impact.

## Conclusion

Do everything you can to protect your app against XSS. If you don't have a good reason to put your JWT in local storage, don't! Default to storing it in a cookie (with the `secure`, `httpOnly` and `sameSite` flags set).

If you do have a good reason to put it in local storage, go for it! Either way, if you take the time to understand what the trade-offs are with the choices you have available to you, you won't stray too far.

## Further Reading

* https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies
* https://www.rdegges.com/2018/please-stop-using-local-storage/ 
* https://dev.to/oktadev/what-happens-if-your-jwt-is-stolen-298d
* Add auth0's article on this
