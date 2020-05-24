---
layout: post
title: So...is my authorization system safe?
subtitle:
tags: [authentication, startups, browser]
comments: false
published: false
---

# Introduction

Let's say that you just bought a new domain for your company named [saaspanda.io](javascript:void()). You are implementing a SaaS product, and you chose a cute animal name for your company, great, orginal!

You quickly implemented a marketing website to present to investors and hosted it on https://saaspanda.io. That's a good start!

Then comes the time to host your prototype and you are already confronted with confusing choices to make: 
  - [https://app.saaspanda.io](javascript:void()) or [https://saaspanda.io/app](javascript:void())?
  - [https://app.saaspanda.io/api](javascript:void()) or [https://api.saaspanda.io](javascript:void())? 
  - Cookies or Local Storage?

You don't have time for this let's pick one and move on! Good choice! You are moving fast and what difference does it make if you spend your time implementing authentication system if your product is not released yet?

Fast forward few months and your product kept growing. You've started adding multiple subdomains. Just to name a few you now have:
  - [community.saaspanda.io](javascript:void()) for your early adopters.
  - [help.saaspanda.io](javascript:void()) for your help center.
  - [stripe.saaspanda.io](javascript:void()) to support your new payement API. You are printing cash now!
  - [developers.saaspanda.io](javascript:void()) for your public API.
  - etc

Today, you decide it is time to stop tampering with the customer database directly and may want to add a [backoffice.saaspanda.io](javascript:void()) for your team to process daily operations related to your business. Or maybe you will buy a separate [backofficesaaspanda.io](javascript:void()) domain for that purpose?

You are looking back at what you've built with your team and you wonder **"so... is my authorization system safe?"**

Disclaimer that we are not going to talk about encryptions here. We'll assume that the encryption is corrent and will take a look from a client side perspective.

Also worth mentioning we are talking about authorizations and not authentication. Authentication means confirming your own identity, whereas authorization means being allowed access to the system.

# Deep dive from an attacker's perspective

The very first thing an attacker will do when doing recon on your application is to find where you store your authentication token. This is what I do during my bug bounty assessments. This is the starting point. This is also the goal.

There are three interesting client-side attacks leveraging authorizations token from a browsers side perspective which we'll take an interest to assess your authorisation system.

#### Cross-side scripting (XSS)
[XSS](https://portswigger.net/web-security/cross-site-scripting) is an attack where the attacker controls an input rendered on your application to execute javascript. When an attacker finds such vulnerability he will want to try and leverage it in multiple ways:
 - steal authorisation token.
 - execute unwanted actions on behalf of the victim such as resetting password, or email, send money, etc...
 - replace the conten of the page.

Depending on your authorisation system, you might be able to reduce the damages.

#### Cross-side request forgery (CSRF)
[CSRF](https://portswigger.net/web-security/csrf) is an attack that forces a user to execute unwanted actions on a website on which they are authenticated. In short, a request to update user password on [https://example.com](https://example.com) could be triggered if they visit [https://evil.com](https://evil.com) **IF** the former doesn't have any CSRF protections in place. 

A CSRF protection can consist of a randomly generated value included in the HTML page that is sent with the request and checked by the server to validate the request.

This one is easy to get wrong.  [INSERT ARTICLE ABOUT HOW TO DO IT RIGHT].
Depending on your authorisation system, you might be able to reduce the damages too.

#### Cross-origin resource sharing (CORS)
[CORS](https://portswigger.net/web-security/cors)



This one too is easy to get wrong [INSERT ARTICLE ABOUT HOW TO DO IT RIGHT].

# Summary



|                |ASCII                          |HTML                         |
|----------------|-------------------------------|-----------------------------|
|Single backticks|`'Isn't this fun?'`            |'Isn't this fun?'            |
|Quotes          |`"Isn't this fun?"`            |<table>  <thead>  <tr>  <th></th>  <th>ASCII</th>  <th>HTML</th>  </tr>  </thead>  <tbody>  <tr>  <td>Single backticks</td>  <td><code>'Isn't this fun?'</code></td>  <td>‘Isn’t this fun?’</td>  </tr>  <tr>  <td>Quotes</td>  <td><code>"Isn't this fun?"</code></td>  <td>“Isn’t this fun?”</td>  </tr>  <tr>  <td>Dashes</td>  <td><code>-- is en-dash, --- is em-dash</code></td>  <td>– is en-dash, — is em-dash</td>  </tr>  </tbody>  </table>      |
|Dashes          |`-- is en-dash, --- is em-dash`|-- is en-dash, --- is em-dash|

marketing: saaspanda.io


XSS on saaspanda.io
  - Cookie theft
  - CORS on saaspanda.io/app
  - CORS on sub.saaspanda.io
XSS on app.saaspanda.io

XSS on subdomain.app.saaspanda.io
