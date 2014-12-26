---
layout: post
title: "Why Building Your Own Backend Can Harm Your Business"
date: 2013-02-15 19:50
comments: true
categories: architecture
---
<img src="/img/post/backend_construction.jpg" width="300" height="300" alt="&copy; http://atom.smasher.org/construction" title="&copy; http://atom.smasher.org/construction" class="right" />
I confess, the headline sounds tremendous as essentially backends are exactly designed for boosting and smoothing your business. Precisely because it is such important, I believe that  it has to be well thought from the outset. I cannot provide an ultimate guideline though I will share some thoughts which are mainly for e-commerce businesses but may also apply for others. I want to make clear that these thoughts are in no way related to my current employer but more what I recall when I think back of the last years.

In the beginning it seems quite fun to implement your own backend. Everyone in the team is excited. You'll start with your favorite framework and javascript libraries. "Let's only implement the most important features, we can deliver the rest later.", you said. You start with simple views for customer, order and backend user management. The first orders start coming in, everyone is happy.  It will not take long until the department heads will start requesting new functionality. 
<!-- more -->
> "Please add number of items, their categories, sizes, colors and barcodes to the grid. Also we need to know if the customer is a new customer or has previously purchased from us before. Latter should show new if the customer has cancelled, rejected or returned that order. We also need a way to export it in CSV."

This will continue with every week. New data, buttons and features are added. You even find out after some while that they actually don't use these buttons and features anymore, as they export the data every morning as CSV and upload it in Google Docs. "It is easier and familiar to use", they will say. What you'll definitely get are complains about performance due to the new columns and joins (particularly that of the export function).

As said backends are crucial for business. If something does not work, a whole department can be left idling, which is likely to happen once or twice a week. You will pause your regular release iteration and focus on solving these issues. While doing this you'll get managers coming by reminding you about the money and hours wasted due to this or that no order could be shipped out today and customs close in 30 minutes.

Another major point is security. Your basic backend user management may lack granular access control levels. Sometimes you remove an export feature for security reasons as requested by management. You forgot that department X was depending on it. Your team is busy with fixing other crucial backend issues that you were barely able to provide a feature to (de)activate users. As there is no search feature HR still needs to find the user by zapping through pages. Occasionally they forget to deactivate a user and integrating LDAP is so deep in your backlog that you forgot it exists.

## Build a loosely coupled architecture using AMQP

Months pass by and the backend becomes more mature. Tons of new functionality have been added. It could be a product editing tool, content management system, voucher or cart rule engines. Once you start thinking that you have reached the plateau, management will summon a meeting to express their wishes to license and integrate expensive [Enterprise Resource Planning (ERP)](http://en.wikipedia.org/wiki/Enterprise_resource_planning) and [Customer Relationship Management (CRM)](http://en.wikipedia.org/wiki/Customer_relationship_management) systems, which basically replaces major stake of your backend's functionality. Beside the fact that you have thousands of invested hours > [/dev/null](http://en.wikipedia.org/wiki//dev/null), we also need to invest hundreds of more hours to make the integration happen as your backend is not designed to support these 3rd party systems.

An alternative approach are [loosely coupled software architectures](http://en.wikipedia.org/wiki/Loose_coupling). Theoretically it means that components can perform tasks independently and unaware of each other. Practically it means that you build your application as logical components communicating with each other on a way which does not require any knowledge about the sender's respectively recipient's nature. Only the communicated piece of information, the message, counts.

This is where [Advanced Messaging Queuing Protocol (AMQP)](http://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol) comes into play. AMQP is basically a message broker handling all the communication between your components in a reliable and secure manner. As said above, only the message counts. From a source code point of view, it can be compared to interfaces. As long as a component publishes and consumes the expected messages, everything is totally fine.

A loosely coupled software is flexible and open for change. Your management's decision to integrate a CRM tool would mean for your team only to make sure that the CRM writes and reads the messages required. It can easily replace your customer management component. Even without such integration initiatives it is super interesting. Imagine your image processing component written in PHP has performance issues. You can easily rewrite it in C.

The great thing about AMQP protocol is that it is an open standard and ensures interoperability. Means even the message broker can be easily replaced without hassles. There are many implementations, even [Microsoft](http://blogs.msdn.com/b/interoperability/archive/2011/10/12/amqp-1-0-specification-now-available.aspx) is in on it. My favorite is [RabbitMQ](http://www.rabbitmq.com/).

I know a company which uses RabbitMQ intensively. Recently they agreed that the programming language and framework they used for their entire product is limiting and causing them frequent operative problems. To evaluate their new language candidate they simply rewrote a small component and deployed it. With the performance and stability results of this they decided to go for it. "Should be a big deal this switch", you might think! Not really, thanks to AMQP, they just rewrite and deploy component by component. This also gave the staff time to learn the new language.

## Use Standard Ware as Early as Possible

This is a point which took me a lot time to realize. As an open source enthusiast I believed that I can do anything better, nicer and more user friendly based on open source solutions. Actually I still believe that I can do it, but the question I ask myself now is: is it worth it?

Your company's staff is a heterogeneous group of different roles and experiences. Every single staff member has bunch of tasks to do coupled with more or less clear expectations by their managers. If they don't have the right tools to accomplish their tasks, if your backend solution prevents them to work efficient or even if the required functionality in your backend is not working, then they can be mad as hops. It is understandable.

Staff most probably won't ever appreciate your open source enthusiasm (Hold on: you use open source technology, but is your backend's source code also open and public? Let me guess, it's not!). They don't care if the button has been designed with CSS3 or the action on click is written with [CoffeeScript](http://coffeescript.org/). The best compliment you'll get is going to be: "Wow cool, it looks like Microsoft Excel!". 

Average Joe will be most comfortable with the tool chain of previous career experiences which is domain related. An HR officer and a finance manager will have experience with different tools except such as Microsoft Office and Windows. It is good practice to provide them domain expertise solutions from the beginning. With today's selection of [Software as a Service (SaaS)](http://en.wikipedia.org/wiki/Software_as_a_service) solutions with proper APIs it is easier to integrate then to implement it yourself. Believe me you don't want to write a finance module!

I asked myself recently, why do businesses not directly start with an ERP and CRM? The management's answer would be most likely: we did/do not have the budget for it. Thinking of solutions such as Microsoft Dynamics or SAP, it is true.

But I think it's not an excuse. There are dozens of open source or low budget solutions out for selection. You may want to have a look at [OpenERP](https://www.openerp.com/en/) for ERP and [pimcore](http://www.pimcore.org/) for CMS and PIM. They might not be the ultimate solutions, but it's better than investing a year in building a backend and then buying an ERP for the same cost as one or two months of in-house development.

## Concentrate on The Customer Experience

With all the trouble for the backend you and your team may loose their focus on the front end. In contrary to the backend, where standard ware is good, customers expect a unique experience in return for their loyalty. Successful businesses have this one thing in common, they have the customer on top of their priority and value list (see [Amazon](http://www.amazon.com/Values-Careers-Homepage/b?ie=UTF8&node=239365011) and [Zappos](http://about.zappos.com/our-unique-culture/zappos-core-values)).

Building a technology and development team is basically a talent hunting. You or your company may have invested so much energy and money to get the web and UX talents on one board. Instead of wasting their expertise and efforts in an ever changing backend, you should let them work on the website. Your customers will appreciate it.

That said, of course customer experience is not only the website. After sales, operation and customer service is crucial, I don't neglect that. I just say they can work better with right tools and an expert team concentrating on them. Give them experts in ERP, CRM and Business Intelligence among with the tools and they'll be awesome. A web engineer will be out of place.

## Conclusion

I admit, this is not a manifesto. I just had these thoughts while looking back at all the years of efforts and work. At the same time I thought how I'd try to do it better if I'd be starting a new e-commerce business.

Even if I won't go for an ERP and CRM from the beginning, I would really consider the loose coupling of the software. I tinker with the idea of using pimcore for years but never had a project to try.

Too complicated? Project or business too small for this? Then get [Shopify](https://app.shopify.com/services/signup?ref=halk) or basic [Magento](http://www.magentocommerce.com/) and live with it.