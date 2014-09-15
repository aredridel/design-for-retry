# So we all know HTTP

2xx OK
3xx Go elsewhere
4xx Go away
5xx ... so About that.

# RETRY

5xx are errors the client can handle

# Causes!

    database down
    bug in a service
    Deploy in progress
    power failure
    kicked a cable
    Network congestion
    Capacity exceeded
    Microbursts

# Need a queue

# GEARMAN

Queues built in

# Two statuses:

OK
FAIL

# Design so FAIL can be retried.

# Automatically tries a background job again.

And again.

And again.

# If it isn't sure it worked?

Tries it again.

# How many of you have used a job queue?

# THERE IS A DISTRIBUTED JOB QUEUE YOU HAVE USED

TRILLIONS of messages

MILLIONS of nodes

100% availability (at least partial) for years.

40 years.

Resilient to MILLIONS of bad actors.

The most malicious network.

# EMAIL.

250 OK
4xx RETRY
5xx Fail

# Responsibility for messages

250 - accept responsibility.
4xx - reject responsibility
5xx - return responsibility

# reject responsibility.

If there's an error? Fail fast.

# No timeouts

Cascade of timeout.
250ms. 1s. Ten services. Ten seconds. Terrible user experience.

# Fail fast. Queue work you can't reject. Reject everything you can if there is an error.

# You need a smart client.

Keeps outstanding requests.
Resubmit.
