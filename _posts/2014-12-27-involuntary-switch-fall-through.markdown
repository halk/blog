---
layout: post
title: "Involuntary Switch Fall-Through (Bug Log #1)"
date: 2014-12-27 23:05
comments: true
tags:
    - coding
---
*There are always bugs in software. Whereas most have only minimal impact at all, [some](http://en.wikipedia.org/wiki/Mars_Climate_Orbiter#Cause_of_failure) are catastrophic. Nevertheless there are a few bugs and failures in a developer’s lifetime which are humorous and memorable. This post is part of a series of failures I personally encountered — not necessarily authored — in my career. The next post in this series is [The Demo Effect](/blog/the-demo-effect).*

A few years ago, I managed an e-commerce platform which has segregated front- and back-end systems. The front-end relies on cached and indexed data provided by the SQL-based back-end system. Beside of a full reindex, a worker is listening to changes in the back-end system. The worker code differentiates the type of the change and calls specific subroutines. This differentiation is accomplished with a `switch` statement like this:

~~~php
switch ($change['type']) {
    case 'cms':
        $this->_updateCms($change['result']);
        // other code
        break;
    case 'product':
        $this->_updateCatalog($change['result']);
        // other code
        break;
~~~

One day we received an incident log reporting a broken payment form on the checkout page. The options for the card type dropdown (usually Visa, MasterCard etc.) was replaced with cryptic options. A full reindex solved the symptom restoring the checkout functionality. Yet an hour or two later (while we are still trying to understand the problem) the symptom appeared back.

As a hands-on manager I dived into the code with the team and little time later nailed the root cause: a missing `break` statement.

~~~php
switch ($change['type']) {
    // ...
    case 'urlrewrite':
        $this->_updateUrlRewriteRule($change['result']);
        // other code
    case 'paymentOptions':
        $this->_updatePaymentOptions($change['result']);
        // other code
        break;
    ...
~~~

As the `$data` variable was used the same way it did not trigger any exceptions or error log entries.

It turned out that someone used a feature of the system which was never used before. Since the core software was provided by the holding company, the relevant code change (which was authored many months before we had this issue) did not go through our review process. The author of the `urlrewrite` case omitted the `break` statement as it is the last case. The author of `paymentOptions` simply added a new case without realising that the previous case is missing the `break` statement. We added `break` and notified the upstream authors.

In general I think a `switch` statement should not be used for this kind of operation.
