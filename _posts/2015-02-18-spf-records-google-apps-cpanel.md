---
layout:     post
title:      "SPF Records for Google Apps on Shared Hosting and Cpanel"
subtitle:   "How to fix the all too common problem of SPF authorization with Cpanel."
date:       2015-02-18 12:00:00
author:     "Ganesh Datta"
comments:   true
---

Using my Google Apps mailservers on a shared hosting server was causing SPF authentication issues – I was getting a **“Received-SPF: neutral (google.com: xxx.xxx.xxx.xxx) is neither permitted nor denied”**. This was causing our emails to end up in the users’ spam folders.

Even after following all kinds of tutorials both from Google and elsewhere on setting up SPF records in Cpanel, nothing was working. After messing around with cpanel and mixing and matching a ton of different methods, I was finally able to get the SPF test to pass.


## Setting up SPF Records on CPanel
The fix to get Google Apps SPF records to cooperate with shared hosting was very simple (assuming that your MX records are all set up correctly, which is outside the scope of this article).

 - Let’s assume that you’re sending email from an account called newsletters@yourdomain.com. Create this account on your Google Apps Admin page, as well as on your CPanel settings. *This is crucial*!
 - Get your **DKIM token** from the Google Apps Admin page by going to Apps > Gmail > Email Authentication. Add this TXT record to your CPanel settings.
 - Then, find out the mailserver being used by your host. An easy way to do this is to send a test email through your website (Wordpress, Drupal, etc), and look at the “original email”, meaning the email including headers and other technical details. 
 - For example, to do this on a Gmail account, click on the arrow on the top right corner of the email, then choose **Show original message**. Find the SPF part of the email, and find where the email is being sent from. In my case (Hostmantis), it was misrv20.fastcpanelserver.com.
 - Add the provided spf record from Google, and then there are two more things that we need to add to it. These are a:*yourdomain.com* and a:*your.mailserver.com*. It should become:

```
v=spf1 +a +mx +a:(your website’s IP address) +include:_spf.google.com +a:(your_mail_server) ~all
```

Once Google Apps says that your email is being authenticated, go ahead and send a test email again. It should say that your spf is now passing. And that’s it!

If that works or even doesn’t, please leave a comment below!

Please visit securaid.com for more tutorials and articles about security.