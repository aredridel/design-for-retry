
# Design for Retry

### Microservices, REST, message busses<br> and why idempotency is the only way to scale

<br><br><br><br><br><br><br><small><a href="http://dinhe.net/~aredridel/">Aria Stewart</a> &middot; <a href="http://twitter.com/aredridel">@aredridel</a></small>

I'm here thanks to [PayPal](http://paypal.com)

# So we all know HTTP

#
2xx OK

3xx Go elsewhere

4xx Tell user what they did wrong

5xx Bail out and log an error

# Error avoidance

## We call this

# You can't avoid errors

## Here's the secret

# Handle errors instead

4xx Tell the user what they did wrong

5xx Save that request and do something with it.

# Retry it

5xx are errors the client can handle

# Causes!

* database down
* bug in a service
* Deploy in progress
* power failure
* kicked a cable
* Network congestion
* Capacity exceeded
* Microbursts

#

* Tree fell on the data center
* earthquake
* tornado
* birds, snakes and aeroplanes
* Black Friday
* Slashdot effect
* Interns
* QA tests
* DoS attack

# You need a queue

# Lots of ways to do it

Database on the nodes

Log file

Queue server

# gearman

Queues built in

There are many alternatives, but `gearmand` is very simple.

# Two statuses:

* OK
* FAIL

# design so FAIL can be retried.

# `gearmand` automatically tries a background job again.

And again.

And again.

# If it isn't sure it worked?

Tries it again.

# How many of you have used a job queue?

# You have used a job queue

# Let me tell you about one

TRILLIONS of messages

MILLIONS of nodes

100% availability (at least partial) for years.

32 years.

Resilient to MILLIONS of bad actors.

The most malicious network.

# EMAIL.

250 OK

4xx RETRY

5xx Fail

# Responsibility for messages

250 - accept responsibility

4xx - reject responsibility

5xx - return responsibility

# reject responsibility.

If there's an error? Fail fast.

The client can retry.

# Fail fast.

Queue work you can't reject. Reject everything you can if there is an error.

# You need a smart client.

Keeps outstanding requests.

Resubmit.

# FIXME
## additional ideas past this point

# No timeouts

Cascade of timeout.

250ms. 1s. Ten services. Ten seconds. Terrible user experience.

