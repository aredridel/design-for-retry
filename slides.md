
# Design for Retry:
## Microservices, REST, and why Idempotency is the only way to scale

I'm [Aria Stewart](http://dinhe.net/~aredridel), that's [@aredridel](http://twitter.com/aredridel) just about everywhere.

I work on the [Kraken.js](http://krakenjs.com) framework.

---

# I'm going to talk about errors.

## It's going to be okay.

^ How many of you look forward to figuring out error handling?

^ That many?

---

```
if (err) {
    alert(err.message);
} else {
    doMyThing();
}
```

^ How many of you have written something like this?

---

# We all know HTTP

^ Every day, I get asked "How do I make a good API out of this?" and after we get through the litany of "identify things with URLs" and "use good verbs that mean the right things", we get into the really hard questions.

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

# But you can't just do things twice?

## We must make operations _idempotent_

---

# Idempotency

## Repeated actions have no effect, give the same result

This means being smart about IDs. Don't recycle!

Check if things are already done.

They are? Just give the same answer again.

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

Database on each node. Maybe LevelDB?

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

^ Not quite true. But two phase commit requires so much coordination that you basically get a "CP" distributed system.

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

Try a different server!

Try a second queue service. Maybe have a fallback plan.

---

# Smart Clients <br> on the device

## Toto, we're not in AWS anymore.

^ We talk about microservices like they're a backend phenomenon, hidden behind some reverse proxy and running on Heroku or AWS

---


# Ever lose an email because you've been logged out?

^ Now, a lot of systems try to avoid this by saving your work periodically.

---

# Latency + Mutable state = Distributed system

## CAP Theorem Applies!

^ It turns out that in any system with latency and mutable state, we're working with a distributed system. This means that our users and their devices are involved. And with a distributed system, CAP applies!

---

# C = Consistency

If there's state that one part knows of that another doesn't? That's inconsistency.

---

# Job queues are controlled inconsistency.

^ "This part of the change hasn't happened yet"

---

# Ever try to write email on the web while not on the Internet?

## It's cloud easy!

^ I biked the pacific coast from Vancouver to San Francisco in 1997. My email was as reliable from a Palm Pilot with a modem as it is from my phone in 2014. Also, I can point out, its total capacity was smaller than `localStorage`

^ Now, sending an email is a particularly easy case: They're almost entirely independent of each other; even if a later message arrives first, there's enough metadata to reconstruct order of messages in a thread, and since it's human-processed, any slop can get resolved by people.

---

# This is really good for offline-first design!

## Being offline is the ultimate retriable error.

---

# Some ideas

---

## Use your queue as a place to measure for system sizing

---

## Queue things in `localStorage`

^ local storage is really good for keeping a queue of requests pending. For single page apps, this is probably the most amazing strategy for promoting a web app into an app app.

---

## Use third-party storage

^ Your app could function even when your servers are completely offline. Host the source in a CDN. Store to your own servers. If they're down, store things to an S3 share with CORS turned on for later replay.

---

## Integrate third-party services with this approach.

^ Anything with CORS turned on is a good candidate. If your app is actually managing state in a deeper way, being somewhat dependent on several external services is a lot less risky.

---

## Use different strategies for available resources vs contended

^ Imagine you're selling widgets with a limited supply. Early on, you know that most orders will be fulfillable. The app starts optimistically, doesn't lock any resources and the user interface tells users it's submitting requests. As stock dwindles, the mode switches: Objects are reserved for a period of time while the order is being built, and requests that have to be queued tell the user to check back later to see if they worked.

---

# Thank you!

I hope you have lots of ideas queued up.

Save your ideas and unspool them onto Twitter. Let me know if this changed how you think about designing applications!
