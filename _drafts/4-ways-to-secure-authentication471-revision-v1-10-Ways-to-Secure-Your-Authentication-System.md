---
id: 473
title: 10 Ways to Secure Your Authentication System
date: 2017-04-26T11:42:27-04:00
author: Sid
layout: revision
guid: https://ducktypelabs.com/471-revision-v1/
permalink: /471-revision-v1/
---
Target it to folks who don&#8217;t like Devise and/or want to roll their own &#8220;simple&#8221; authentication

People who hate Devise and want to use another gem People who hate Devise and want to roll their own People who are fine with Devise People who are not sure either way

Authentication frameworks can be somewhat of a contentious topic. Take Devise, one of the most popular options, for example. Critics of Devise point out, perhaps rightly so, that there is a lack of clear documentation, that it is hard to understand, hard to customize and wonder if we wouldn&#8217;t be better off using a different gem or even rolling our own custom authentication system. Advocates of Devise, on the other hand, point out that Devise is the result of years of expert hard work and that by rolling your own, you&#8217;d be foregoing much of the security that comes with using a respected and highly audited piece of software. If you&#8217;re new to this conversation, you might ask yourself why you would even need Devise if you can just use `has_secure_password` and expire your random password reset tokens within a short enough time-period. If you&#8217;ve just completed the venerable Hartl Rails tutorial, or watched the equally eminent RailsCast on implementing an authentication system from scratch (link), you might be wondering why people jump straight to Devise or if there&#8217;s something inherently insecure with the systems in these tutorials.

Like in most areas of software (and life), the answer is that it depends.

Whether you&#8217;re sick of how &#8220;bloated&#8221; Devise can be and want to roll your own authentication system, or you&#8217;re perfectly fine with it and

You might think that Devise

  1. Use separate tables for passwords, tokens etc (credit to Jeremy Evans and Rodauth)
  2. Read authentication libraries (Devise, Authlogic, Clearance, Rodauth etc)
  3. make tokens user specific to mitigate bruteforcing
  4. Use something like Rack::Attack
  5. Set up your security headers correctly
  6. Consider not using a cookie to store session info and in something like Redis instead (credit to Ali Najaf) 
  7. Logging
  8. Attack your auth system
  9. Use secure compare wherever it applies
 10. Learn about security and secure other areas of your app (shameless plug here)
 11. Enforce password length
 12. RNG