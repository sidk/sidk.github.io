---
id: 488
title: 4 Ways to Secure Your Authentication System in Rails
date: 2017-06-16T12:22:44-04:00
author: Sid
layout: revision
guid: https://ducktypelabs.com/471-revision-v1/
permalink: /471-revision-v1/
---
Authentication frameworks in Ruby on Rails can be somewhat of a contentious topic. Take Devise, one of the more popular options, for example. Critics of Devise point out, perhaps rightly so, that there is a lack of clear documentation, that it is hard to understand, hard to customize and wonder if we wouldn&#8217;t be better off using a different gem or even rolling our own custom authentication system. Advocates of Devise, on the other hand, point out that Devise is the result of years of expert hard work and that by rolling your own, you&#8217;d forego much of the security that comes with using a highly audited piece of software. If you&#8217;re new to this conversation, you might ask yourself why you would even need Devise if, like in the Hartl Rails tutorial or the RailsCast on implementing an authentication system from scratch, you can just use `has_secure_password` and expire your password reset tokens (generated with `SecureRandom`) within a short enough time-period.

Is there something inherently insecure in the authentication systems described in the Hartl tutorial and the RailsCasts? Should a gem like Devise be used whenever your app needs to authenticate its users? Does using a gem like Devise mean that your authentication is now forever secure?

Like in most areas of software, the answer is that it depends. Authentication does not exist in isolation from the rest of your app and infrastructure. This can mean that even if your authentication system is reasonably secure, weaknesses in other areas of your app can lead to your users being compromised. Conversely, you might be able to get away with less than optimum security in your authentication system if the rest of your app and infrastructure pick up the slack.

The only way you, an app developer, can answer these questions satisfactorily is to deepen your understanding of security and authentication in general. This will help you as you make the tradeoffs that will inevitably arise when you are building and/or securing your app.

Whether you want to use Devise (or a similar third party gem like Clearance) or roll your own auth, here are 4 specific ways you can make authentication in your app more secure. Though your mileage may vary, I hope at the very least one of them gives you something to think about.

# Throttle Requests

The easiest thing an attacker can do to compromise your users is to guess their login credentials with a script. Users won&#8217;t always choose good passwords, so given enough time, an attacker&#8217;s script will likely be able to compromise a significant number of your users just by making a large number of guesses based on [password lists](https://github.com/danielmiessler/SecLists/blob/master/Passwords/10_million_password_list_top_100.txt).

You can make it hard for attackers to do this by restricting the type and number of requests that can be made to your app within a predefined time period. The gem `rack-attack`, for example, is a Rack middleware that gives you a convenient DSL with which you can block and throttle requests.

Let&#8217;s say you just implemented the Hartl tutorial and you now want to add some throttling, you might do something like this after installing `rack-attack`:

    throttle('req/ip', :limit => 300, :period => 5.minutes) do |req|
      req.ip
    end
    

The above piece of code tells `Rack::Attack` to limit any given IP to at most 300 total requests every 5 minute period. You&#8217;ll notice that since the block above receives a request object, we can technically throttle requests based on any arbitrary request parameter.

While this limits a single IP from making too many requests, attackers can get around this by using multiple IP&#8217;s to bruteforce a user&#8217;s account. To slow this type of attack down, you might consider throttling requests per account. For example:

    throttle("logins/email", :limit => 5, :period => 20.seconds) do |req|
      if req.path == '/login' && req.post?
        req.params['email'].presence # this will return the email if present, and nil otherwise
      end
    end
    

If you&#8217;re using Devise, you also have the option to &#8220;lock&#8221; accounts if there are too many unsuccessful attempts to log in. You can also implement a lockout feature by hand. However, there is a flip side to locking and/or throttling on a per-account basis &#8211; attackers can now restrict access to arbitrary user accounts by forcing them to lock out, which is a form of a &#8220;Denial of Service&#8221; attack. In this case, there is no easy answer. When it comes to choosing security mechanisms for your app, you&#8217;ll have to decide how much and what type of risk you&#8217;re willing to take on. [Here is an old StackExchange post that might be a good starting point for further research](https://security.stackexchange.com/questions/487/why-do-sites-implement-locking-after-three-failed-password-attempts)

A note about `Rack::Attack` and DDoS attacks: Before actually implementing a throttle, you&#8217;ll want to use `Rack::Attack's` &#8220;track&#8221; feature to get a better idea of what traffic looks like to your web server. This will help you make a more informed decision about throttling parameters. [Aaron Suggs](https://github.com/ktheory), the creator of Rack::Attack, says it is a complement to other server-level security measures like `iptables`, `nginx limit_conn_zone` and others.

DoS and DDoS attacks are vast topics in their own right so I&#8217;d encourage you to dig deeper if you&#8217;re interested. I&#8217;d also recommend looking into setting up a service like Cloudflare to help mitigate DDoS attacks.

# Set up your security headers correctly

Even though you probably serve requests over HTTPS, you might be vulnerable to a particular type of Man in the Middle attack known as SSL Stripping. To illustrate, imagine you lived in Canada and bank with the Bank of Montreal. One day, you decide to email money to someone and type &#8220;bmo.com&#8221; into the address bar in Chrome. If you had dev tools open and on the &#8220;Network&#8221; tab, you&#8217;d notice that the first request to bmo.com is made via http, instead of https. A suitably situated attacker could have intercepted this request and begun to serve you a spoofed version of the BMO website (making you divulge login information and what not), and they&#8217;d have been able to do this only because your browser used http, instead of https.

The `Strict-Transport-Security` header is meant to prevent this type of attack. By including the `Strict-Transport-Security` header (aka the HTTP Strict Transport Security or HSTS header) in its response, a server can tell a browser to only communicate with it via https. The HSTS header usually specifies a `max-age` parameter and the browser equates the value of this parameter to how long it should use https to communicate with the server. So, if `max-age` was set to `31536000`, which means one year, the browser would only communicate with the server via https for a year. The HSTS header also lets your server specify if it wants the browser to talk via https on its subdomains as well. See [here](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security) & [here](https://www.troyhunt.com/understanding-http-strict-transport/) for further reading.

To make this happen in Rails, do `config.force_ssl = true`. This will ensure that the HSTS header is set to a value of `180.days`. To apply https to all your subdomains, you can do `config.ssl_options = {hsts: {subdomains: true}}`.

The loophole in this is that the first ever request to the server might still be made via http. The HSTS header protects all requests except the first one. There is a way to have the browser always use https for your site, even before the user has actually visited, and that is by submitting your domain to be included in the [Chromium preload list](https://hstspreload.org/). The &#8220;disadvantage&#8221; with the preload approach is that you will never ever be able to serve info via http on your domain.

Having HSTS enabled doesn&#8217;t mean your users will be absolutely safe, but I&#8217;d wager they&#8217;d be considerably safer with it than without.

If you&#8217;re curious, you can quickly check what security related headers (of which HSTS is one) your server responds with on [securityheaders.io](https://securityheaders.io). I advise looking into all the headers here to learn and decide if they apply to your situation or not.

# Read authentication libraries (Devise, Authlogic, Clearance, Rodauth and anything else you have access to)

This especially applies if you&#8217;re rolling your own, but even if you don&#8217;t you can learn a lot from how another gem does a similar thing. You don&#8217;t always have to read the source code itself to learn. The change logs and update blog posts from maintainers can be just as informative, because they often go into detail about vulnerabilities that were discovered and the steps taken to mitigate them. Here are three things I learned from Rodauth and Devise that you might find intriguing:

### Restricted Password Hash Access (Rodauth)

Unlike Devise and most &#8220;roll your own auth&#8221; examples, Rodauth uses a completely separate table to store password hashes, and this table is not accessible to the rest of the application. Rodauth does this by setting up two database accounts, `app` and `ph`. Password hashes are stored in a table that only `ph` has access to, and `app` is given access to a database function that uses `ph` to check if a password hash matches a given account. This way, even if an SQL injection vulnerability exists in your app, an attacker will not be able to directly access your users&#8217; password hashes.

### User Specific Tokens (Rodauth)

Rodauth not only stores password reset and other sensitive tokens in separate tables, it also prepends every token with an account ID. Imagine your forgot password link looked something like &#8216;www.example.com/reset\_password?reset\_password_token=abcd1234&#8217; and an attacker was trying to guess a valid token. The attacker&#8217;s guess could potentially be a valid token for any user. If we prepend the token with an account ID (maybe the token looks like `reset_password_token=<account_id>-abcd1234`), then the attacker can only attempt to brute force their way in to one user account at a time.

### Digesting Tokens (Devise)

Devise, since version 3.1 came out a few years ago, digests password reset, confirmation and unlock tokens before storing them in the database. It does so by first generating a token with `SecureRandom` and then digesting it with the `OpenSSL::HMAC.hexdigest` method. In addition to protecting the tokens from being read in the event that an attacker is able to access the database, digesting tokens in this manner also protects them against timing attacks since it would be near impossible for an attacker to control the string being compared enough to make byte-by-byte changes.

If you want to know more about Rodauth, [check out their github page](https://github.com/jeremyevans/rodauth) and also watch [this talk](https://www.youtube.com/watch?v=z3HZZHXXo3I&feature=youtu.be) by Jeremy Evans, its creator.

To summarize, the more you know about how other popular authentication frameworks approach authentication and the steps they take to avoid being vulnerable to attack, the more confident you can be in assessing the security of your own authentication set up.

# Secure the Rest of Your App

Authentication does not stand in isolation. Vulnerabilities in the rest of your app have the potential to bypass any security measures you might have built into your authentication system.

Let&#8217;s consider a Rails app with a Cross Site Scripting (XSS) vulnerability to illustrate. Imagine the XSS vulnerability exists because there&#8217;s an `html_safe` in the codebase somewhere that unfortunately takes in a user input. Now, because our app is a Rails (4+) app, we have the `httpOnly` flag set on our cookie by default, which means any Javascript an attacker is able to inject won&#8217;t have access to `document.cookie`. Though it might seem like our app is safe from session highjacking attacks, our attacker can still do a bunch of things that compromise a user&#8217;s session. For example, they can inject Javascript that makes an AJAX request to change the user&#8217;s password. If the password change form requires the current password, they can try to change the user&#8217;s email (to their own) and initiate a password reset flow.

In short, an XSS vulnerability sort of makes it irrelevant how secure your authentication is, and the same can be said of other vulnerabilities like [Path Traversal](https://www.owasp.org/index.php/Path_Traversal) or CSRF.

Learning about security vulnerabilities and then applying that knowledge to attack your own app is a great way to, over the long term, write more secure code. I&#8217;d also encourage you to read through resources like the [Rails Security Guide](http://guides.rubyonrails.org/security.html) and security checklists like [this](https://github.com/eliotsykes/rails-security-checklist) and [this.](https://github.com/brunofacca/zen-rails-security-checklist)

# Conclusion

_Shameless Plug:_ If you&#8217;re within a few hours flight from Toronto, I&#8217;d love to come talk to you and your team about security, free of charge. Get in touch with me at sidk(AT)ducktypelabs(DOT)com for more info.

The above list is not meant to to be comprehensive, and what I&#8217;ve left out can probably fill multiple books. However, I hope I&#8217;ve given you a few things to think about and that you&#8217;re able to take away at least one thing that will make your app more secure.

I&#8217;d love to hear from you! Post in the comments section below &#8211; what do you do to secure your auth?

<div id="mc_embed_signup" style="background: #dff7fe; padding: 15px;">
</div>