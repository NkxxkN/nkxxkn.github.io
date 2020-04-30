---
layout: post
title: Admin privilege escalation using parameter pollution
subtitle: 
tags: [write-up, parameter pollution, high, takeaways, $1500]
comments: false
published: true
---

### Background

**Parameter pollution** is an attack in which an attacker modifies a request made to the server to add an extra parameter that was not intended to be provided with the request but that does get taken into account by the server/database. 
Parameter pollution can happen in GET parameters request. For instance sending... 
```
GET /reset_password?email=victim@gmail.com&email=attacker@gmail.com
```
...hoping that the reset password link for `victim@gmail.com` would be sent to `attacker@gmail.com`.

In this write up we are going to talk about a case of JSON parameter pollution.

**The vulnerable application** is a SaaS company. It gives access to different "folders" to its users which can be public or private. Folders to this application are like repositories for Github, boards for Trello  or channels for Slack. The application provides following levels of permission within an organisation:
  - Admins: All privileges
  - Users: Can read/write all public folders. Can read/write private folders they are invited to.
  - Read only: Can read all public folders. Can read private folders they are invited to.
  - Guests: Can read/write public folders they are invited to.

Ideal scenario is to gain admin privileges as a guest. This is what I always aim for in this program. Read only or member to admin is great too!

### Description

I was playing with the "Invite Guest to a folder" feature which allows a folder owner to invite a guest from another company to collaborate on a particular folder. 

The request looked like:

```
POST /folders/123xyz/invite HTTP/1.1
Host: app.redacted.com

{"admin":false,"member":{"email":"guest@gmail.com","role":"guest"}}
```

And the response:

```
{
  "id":12345,
  "email":"guest@gmail.com"
  "is_pending":true,
  "is_admin":false,
  "is_guest":true
}
```

First thing I tried was to change the request value for `admin` from `false` to `true`. It did work when I tried this while logged in as an admin but that's intended behavior. It did not work when I was logged in as a regular user though.
Another idea I had was to change the `role` from `guest` to `admin` or `member` but that did not work either.

I did not stop here.

My next idea was to look at the response and try simple parameter pollution. 
I added `"is_guest":false` to the request payload and hoped that it would be reflected but once again it did not.
Success came when I decided to try with following request adding `"is_admin":true`, leaving `"role":"guest"` and `admin:"false`.

The successful request looked like:

```
POST /folders/123xyz/invite HTTP/1.1
Host: app.redacted.com

{"admin":false,"member":{"email":"guest2@gmail.com","role":"guest", "is_admin":true}}
```

And the response:

```
{
  "id":12346,
  "email":"guest2@gmail.com"
  "enabled":true,
  "is_pending":true,
  "is_admin":true,
  "is_guest":false
}
```

I went to my emails, registered the account and was logged in as an admin as expected. I had a privilege escalation from regular user to admin. 

Now remember that ideal scenario is to get privilege escalation from read only to admin, or even better from guest to admin.
I tried to send guest invites from a guest but that was not allowed because a guest can not be owner of a folder. I knew a read only user could be owner of a folder. So in the end, I had privilege escalation from read only user to admin within an organisation.


**What happened? Why did this work?**

I can only guess that since there are multiple endpoints to invite new users, create new accounts, update profile etc... backend controllers are using  a shared service to interact with user properties. This is a principle as a developer to not repeat yourself (DRY) in order to keep consistency. That can lead to these kind of vulnerabilities where `is_admin` may be available in other circumstances, hence I was able to send the extra parameter hoping that the backend controller would not filter it out before passing it to the underlying service in charge of creating the user. 

Here is some pseudo code of what the controller might have looked like:
```
def invite_guest(params):
  // If user is admin and admin param is true then make new user admin.
  if (params[:admin] == true && current_user.is_admin == true)
    params[:member][:is_admin] = true

  // Use Users generic service to create new user.
  Users.createOrUpdateUser(params[:member])
end
```

Note that from a developer point of view, this code looks farely safe as the `current_user` needs to be an admin to make the new user an admin too. This is a good example of why you should always clean your user provided data before passing it down to your ORM/database. As a matter of fact, this is generally not a good idea to pass the `params` attribute from the controller to another service. A better idea is to reconstruct an object keeping only expected values.


### Takeaways

  - Familiarize yourself with the technologies. For instance, I noticed that Ruby on Rails application are more likely to be vulnerable to parameter pollution than other back end technologies and will often spend more time looking for this type of vulnerability on RoR applications.

  - Make sure you understand all the roles properly. Another hacker could have easily missed that read only users who are also owner of a folder could send guest invites.

### Summary

 - **Weakness**: Parameter pollution
 - **Reward**: 1500$