---
id: 130
title: Using scope with associations
date: 2015-11-23T14:09:01-05:00
author: Sid
layout: revision
guid: http://ducktypelabs.com/128-revision-v1/
permalink: /128-revision-v1/
---
When you have a `belongs_to` or a `has_one/has_many (with or without :through)` association defined on your model, you will in many situations benefit from defining scopes in your model that reference your association. Defining a custom scope which references attributes in your association table(s) has the potential to DRY up your code and make code in other parts of your application easier to understand.

However, implementing such scopes can cause much confusion and frustration; especially when you see hard to interpret `SQL-y` errors being returned, and have no idea how to go about fixing them.

Referencing attributes of the parent object in the scope

Referencing attributes of the association table in the scope (have to use SQL)

Referencing attributes of any table (need not be an association)

Troubleshooting error messages:  
Remember that a custom scope is the same as defining a class method. So if you&#8217;re stuck, ask yourself how you would implement the scope as a class method.