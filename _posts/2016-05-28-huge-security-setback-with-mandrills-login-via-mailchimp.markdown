---
layout: post
title: "Huge Security Setback with Mandrill's Login via MailChimp"
date: 2016-05-28 14:25:00
comments: true
tags:
    - service
---
*This post has been written and published at 35,000 ft altitude over the [Nicobar Islands](https://en.wikipedia.org/wiki/Nicobar_Islands), Indian Ocean, while enjoying in-flight Internet connectivity on my Emirates flight from Dubai, UAE, to Jakarta, Indonesia.*

Being a [MailChimp](http://mailchimp.com) customer, it was almost obvious to use [Mandrill](http://mandrill.com) for transactional emails. In fact the MailChimp integration we had with our eCommerce platform, came with ready-to-use Mandrill support. I like its APIs and interface. Especially, the test API keys allow running Mandrill on test mode for pre-production environments which practically means that no actual emails are sent out but listed on the app interface. This is another barrier against sending unwanted emails in pre-production - first barrier should be sanitising email addresses along with other customer details in database backups.

From day zero of using Mandrill, my single aversion was to its authentication system. It only had an email address and password login with no multi-login support. This means I had to share my login and password with folks from the rest of the company, from customer service to product development. This was a huge security problem as anyone of them - if deliberately or as a victim of a hack, could hijack the account temporarily and send emails in our name. Imagine a phishing attack sent from the official account.

Earlier this year MailChimp [announced](https://mandrill.zendesk.com/hc/en-us/articles/217467117) that they cease Mandrill as a standalone service and treat it as an add-on to MailChimp. This meant that, after merging your MailChimp and Mandrill account, you could only login via MailChimp. *"Perfect!"*, I thought as MailChimp has multi-user support. That euphory turned into speechlessness quickly after I tried giving access to the first person. MailChimp has multiple permission level for users and I started with the lowest level but learned that a user has to be an admin in MailChimp to get access to Mandrill. I felt that this is actually worse than previously. Now I had to give admin access to MailChimp with even more sensitive information, too.

I hope that the MailChimp team finds a solution to this. I would suggest implementing counterparts for the MailChimp permission levels in Mandrill. In other words, instead of having an admin-or-nothing access, e.g. a *viewer* would only view email activity, *editor* could edit templates and an *admin* could do configuration changes.