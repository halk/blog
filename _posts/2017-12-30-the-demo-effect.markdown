---
layout: post
title: "The Demo Effect (Bug Log #2)"
date: 2017-12-30 15:15
comments: true
tags:
    - coding
---
*This is the second post in the bug log series in which I share some of the humorous moments in my career. The previous post was [Involuntary Switch Fall-Through](/blog/involuntary-switch-fall-through.html)*.

I have neglected the blog for a long time, yet I decided to not let 2017 pass without a post. So here it goes.

This incident has happened eight years ago when I was working for Konexxo. The client was a B2B business group operating in the health and beauty products sector. One of their products was individualised perfume production. We developed an online perfume designer allowing amongst other things to mix the fragrance, choose various flask and packaging options as well as design labels. Upon successful checkout, the production order was then pushed to a 63-metre fully-automated perfume machine. This machine had many stations such as inserting selected flask, printing labels, filling the flask, putting sprayer and cap on, putting into packaging and laser engraving packaging.

<img src="/img/post/perfume_machine.png" width="300" height="166" class="right" style="height: 166px !important" />
During the implementation phase, a potential buyer of the machine design wanted visited the plant and we were tasked with doing a test run. We had a very experimental form which we used for testing purposes. We entered the wished perfume configuration into the form and submitted. The form reloaded and reset all fields. We panicked and did not want to ask for their configuration again. So, we tried to remember the configuration and probably wildly guessed most of it in that process. A whole group now followed the production run. They were just in front of the fragrance fillers, bent down to have a closer look when they were welcomed with a shocking splash of fragrance. Luckily, there was a window in between so that they did not get wet but were startled nonetheless. I am sure this was what put off the buyer.

Neither our form nor the software running the machine did not validate the filling amounts allowing the sum to be larger than the flask can hold. My colleague and I were embarrassed of course but that did not stop that the back-end logic of that experimental form ended up in production. We even named the class *Moppelkotze* - a horrible stew popular in North Germany. It is an unwritten programming law that workarounds and hacks turn out to be permanent.
