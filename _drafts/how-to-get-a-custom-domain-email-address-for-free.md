---
layout: post
title: How to get a custom domain email address for free
excerpt: 'I recently discovered that it''s possible to combine any domain that you
  own with your Gmail account and a free MailGun account to get a free custom domain
  email address!  Here''s how:'
tags:
- custom domain
- mailgun
- website
- email

---
I recently discovered that it's possible to combine any domain that you own with your Gmail account and a free MailGun account to get a free custom domain email address!

When you're done following the below instructions you'll be able to send and receive emails addressed to you@yourdomain.com directly from your Gmail inbox.

# Step 1 - Buy a domain

![](/media/2020-01-14-namecheap logo.png)

Okay, so you still have to pay to own your domain, but the rest is free. And if you want a custom domain email address, then chances are you're interested in your professional reputation. Owning your own domain is a good idea even if you don't plan to start your own sure just yet. I held off on it a couple years and that was enough for a law student to grab zainrizvi.com -_-

I prefer to buy my domains through namecheap.com, though pretty much any company would work. Use whatever you like, but don't let the decision of which site to use to stop you from buying your domain! (Use namecheap.com if you're uncertain)

# Step 2 - Sign up for a [mailgun.com](http://mailgun.com) account

![](/media/2020-01-14-mailgun logo.png)

This is the secret sauce. We'll use MailGun to forward any email sent to your domain straight to your Gmail inbox (umm...you should sign up for Gmail too if you haven't already)

Go to their site at http://www.mailgun.com and sign up.  It's free (even if they want you to give them your credit card)

# Step 3 - Register your domain with MailGun

Within MailGun find the option to add a new domain. Chances are theyll have setup their new user page to guide you through that exact process. Follow the instructions on the page to setup your domain.

Note: MailGun requires a credit card before you can add your domain, but they won't charge you anything if you follow the steps in this blog.

At some point they will ask you to update your domain's DNS records. It seems scary but they walk you through the worst of it.

Then wait for MailGun to confirm that everything is setup correctly (it tends to take less than a hour, but could be longer)

# Step 4 - Setup mail forwarding within MailGun

Within MailGun go to the Receiving section ([https://app.mailgun.com/app/receiving/routes](https://app.mailgun.com/app/receiving/routes "https://app.mailgun.com/app/receiving/routes")) and create a new route.

Set the "Expression Type" to "match recipient" and then for the recipient enter the exact email address you'd like to have (yourname@yourdomain.com). Ensure the checkbox under "forward" is checked, and enter your gmail address there.

Once you hit save, any emails that get sent to yourname@yourdomain.com will get forwarded to your Gmail. (The only exception to this is emails you send from the same Gmail account that the message is being forwarded to. Gmail tries to be smart and hides that message from you. So if you're testing it out, send the message from a different email address).

# Step 5 - Tell Gmail to send messages from your custom address

Gmail has this handy feature called "Send Mail As" which we'll be taking advantage of here.

In Gmail, go to settings->xyz

Click add new

If you want to make this new email address your primary one, select the "make this my default address" button

# And Enjoy!

You're done! Now you've added email to your custom domain at no additional cost, and you get to keep using the wonderful Gmail interface.

Did you find this useful? I'd love to hear it! Drop a comment below or send me a message at my-first-name @ zainrizvi.io.