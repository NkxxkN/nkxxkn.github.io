---
layout: post
title: Admin privilege escalation using XSS
subtitle: 
tags: [write-up, XSS, high, takeaways, $1500]
comments: false
published: true
---

### Background

**Cross site scripting attack** is an attack in which an attacker controlled input is refelcted on a website to execute javascript code in hte context of the victim's browser.

**The vulnerable application** has different levels of permission within an organisation:
  - Admin
  - Member
  - Guest

Ideal scenario is to gain admin privileges as a guest. This is what I always aim for in this program.

### Description

I had been browsing on the application at least 5 minutes a day for the past few weeks without really digging deep when I noticed that a new rich text editor had been released in a hidden part of the application.

I tried using the "Add link" feature:


![Add Link Feature](/img/xss_rich_text.png){: .center-block :}

It was reflected as follow:


![Html](/img/reflected_xss.png){: .center-block :}

Great! Now what can I do with it? Can anyone create the XSS payload? Can anyone see the stored XSS?

No!

Only admins and member can. That mean a privilege escalation scenario is only possible from member to admin. But how?

I tried to steal the authentication cookie first but quickly realised after logging `document.cookie` that the only token that interested me had the HTTP only flag. That means that it can not be accessed with javascript.

Can I update the user password without knowledge of current password? No. What about the email ? No.

Account takeover is note possible then, but privilege escalation still is. I crafted a payload to send the XHR request that updates the role of a user from member to user.
Here is the final payload:

```javascript
javascript:var req = new XMLHttpRequest();req.open('POST','/update/roles/',true);req.setRequestHeader('Content-type','application/json');req.setRequestHeader('X-Requested-With','XMLHttpRequest');req.setRequestHeader('X-CSRF',document.head.querySelector('meta[name="csrf-token"').content);req.send(JSON.stringify({'members':[1337],'admin':true}))
```

I also provided a proof of concept as a video as a I always do with my reports for two reasons:
  - It helps platform triagers to validate the bug faster.
  - In case the issue is fixed before the report was triaged, I have a proof that I can leverage to make sure I get rewarded.

Triager said that the usual payout for XSS only allowing privilege escalation from member to admin are 500$. Because of the quality of my report and the actual proof of concept of the privilege escalation, the triager rewarded me with a nice 1500$ admitting that this new feature was slowly getting rolled out and would have been accessible by guests user later on.

### Takeaways

  - Familiarize yourself with the application you are hacking on. Spend time just browsing and will become an expert at identifying any visual changes. The feature had been released for 4h only when I found the vulnerability.

  - When you find an XSS, this is just the beginning. You might have spent days to find it. Do yourself a favor, spend another hour or two to prove its impact, it will maximize your reward. In this case it tripled it.
    
    If you are worried about duplicates, you can make an initial report with all useful information and potential impact **BUT** make sure you actually come back to it. In addition to maximizing the bounty, those extra hours are not wasted because you always learn a lot doing a real world proof of concept. This time I learnt about HTTP Only Cookies.

### Summary

 - **Weakness**: Stored XSS
 - **Reward**: 1500$