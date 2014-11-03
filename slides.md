
# Design for Retry

### Microservices, REST, message busses<br> and why idempotency is the only way to scale

[Aria Stewart](http://dinhe.net/~aredridel)

[@aredridel](http://twitter.com/aredridel)

I'm here thanks to [PayPal](http://paypal.com)

---

I'm going to talk about errors.

It's going to be okay.

^ I told my last employer that I wasn't going to take a pager. I told them that anything I design would be designed to stay up.

---

# We all know HTTP

^ We all know HTTP. However, every day, I get asked "How do I make a good API out of this?" and after we get through the litany of "identify things with URLs" and "use good verbs that mean the right things", we get into the really hard questions.

^ Because I'm one of those annoying people who answers every question with a question, I always ask "How are you going to handle errors?"

---

2xx OK

3xx Go elsewhere

4xx Tell user what they did wrong

5xx Bail out and log an error

## I'd call this

# Error avoidance

^ There's just one problem with this scheme.

---

# You can't avoid errors

^ You've met the fail whale, you've seen the a big page reading "Sorry!", you've probably seen Varnish's "Guru Meditation" numbers. While that's a really nice shout-out to the Amiga Workbench, it doesn't actually help our our users.

^ You can get a lot of mileage out of 'error avoidance' if your wallet is big enough. Throw ten times as much hardware at it. Hire a dozen more people to be on call. Set up an elaborate pager routine. Get some really expensive load balancers and hire consultants to set all that up for you.

---

## Here's the secret

# Handle errors instead

^ Save an order of magnitude on equipment costs. No, really.

---

4xx Tell the user what they did wrong

5xx Save that request and do something with it later.

^ 4xx errors are really the ones you throw up your hands on. Security faults, access denied, malformed requests. There's no sensible way to continue with most of these.

^ But the server errors, those are the interesting ones.

---

# Retry it

5xx are errors the _requestor_ can handle

^ Let that sink in. We can actually do something with 500 errors. We've got storage, we can journal it to disk and at least say "We'll do our best!"

---

# Causes!

* database down
* bug in a service
* Deploy in progress
* power failure
* kicked a cable
* Network congestion
* Capacity exceeded
* Microbursts

---

* Tree fell on the data center
* earthquake
* tornado
* birds, snakes and aeroplanes
* Black Friday
* Slashdot effect
* Interns
* QA tests
* DoS attack

^ This is where I tell you that the weirdest error I ever had was rock dust from an exposed rock wall in a basement data center was infiltrating the first rack of systems, and we ended up with a fine layer of it all over the motherboard. It made the RAM in that system unreliable. But only a few bits at a time. The hard drive controller hated this and would just go AWOL for a minute or so at a time.

---

# You need a queue

^ Or a heap of saved requests if you can avoid ordering. You are lucky if so!

---

# Lots of ways to do it

Database on the nodes

Log file

Queue server

---

# gearman

Queues built in

There are many alternatives, but `gearmand` is very simple.

The memcache of job queues.

^ This is where I tell you about one of my favorite job queues. I really only like it because it's super simple, and it doesn't guarantee too much. In fact, it has some really interesting limitations.

---

# Three statuses:

* OK (Like 200)
* FAIL (Like 400)
* ERROR (Like 500)

---

# design so ERROR can be retried.

^ This is where I tell you that if you tell the user "Sorry!", you've just said "The dog ate my homework."

---

# `gearmand` automatically tries a job ERROR again.

And again.

And again.

---

# If it isn't sure it worked?

Tries it again.

---

# You _cannot_ know if an error is a failure.

---

# Error handling gets simpler

* Exception? ERROR.
* Database down? ERROR.
* Downstream service timeout? ERROR.

Maybe you retry right away.

---

# How many of you have used a job queue?

---

# You have used a job queue

---

# Let me tell you about one

TRILLIONS of messages

MILLIONS of nodes

100% availability (at least partial) for years.

32 years.

Resilient to MILLIONS of bad actors.

It is attached to the most malicious network.

---

# EMAIL.

250 OK

4xx RETRY

5xx Fail

---

# Responsibility for messages

250 - accept responsibility

4xx - reject responsibility

5xx - return responsibility

---

# reject responsibility.

If there's an error? Fail fast.

The requester can retry.

---

# Fail fast.

Queue work you can't reject. Reject everything you can if there is an error.

---

# You need a smart client.

Keeps outstanding requests.

Resubmit.

---

# We're Nodevember, We're Special

## Bonus content time!

^ I've got ten more minutes up here than I'd planned this talk for, so bonus material time!

---

# Smart Clients <br> on the device

## Toto, we're not in AWS anymore.

^ We talk about microservices like they're a backend phenomenon, hidden behind some reverse proxy and running on Heroku or AWS

---

# Ever lose an email because you've been logged out?

^ Now, a lot of systems try to avoid this by saving your work periodically.

---

# Ever try to write email on the web while not on the Internet?

^ I biked the pacific coast from Vancouver to San Francisco in 1997. My email was as reliable from a Palm Pilot with a modem as it is from my phone in 2014. Also, I can point out, its total capacity was smaller than `localStorage`

---

# Some ideas

* Queue things in `localStorage`
* Use third-party storage
* Integrate third-party services with this approach

---

# This is really good for offline-first design!

## Being offline is the ultimate retriable error.
