---
node_id: 3657
title: Setting Up a Producer-Consumer Model with Cloud Queues
type: article
created_date: '2013-08-21'
created_by: Megan Meza
last_modified_date: '2016-01-11'
last_modified_by: Rose Contreras
product: Cloud Queues
product_url: cloud-queues
---

Setting up a Producer-Consumer model in Cloud Queues consists of posting
messages to your queue, consumers claiming messages from that queue, and
then deleting the completed message.

The producer-consumer mode has the following characteristics:

-   Messages are acted upon by one (and only one) worker.
-   Worker must delete message when done.
-   TTL restores message to unclaimed state if worker never finishes.
-   Ideal for dispatching jobs to multiple processors.

This mode is ideal for dispatching jobs to multiple processors.

### Posting messages to a queue

Queues support posting 10 messages at the same time, so lets try to post
two within the same request.

***Request***

        $ curl -i -X POST https://$ENDPOINT:443/v1/queues/samplequeue/messages \
          -d '[{"ttl": 300, "body": {"event": "two"}}, {"ttl": 60, "body": {"event": "three"}}]' \
          -H "Content-type: application/json" \
          -H "Client-ID: e58668fc-26eb-11e3-8270-5b3128d43830" \
          -H "X-Auth-Token: $TOKEN"

**Note:** Use Claim TTLs to protect from agent/client failure while they have a message claimed.  In the event that a server goes offline and cannot complete the message it has claimed, the claim TTL will expire and that message will be returned to the queue for other consumers or workers to claim.

***Response***

        HTTP/1.1 201 Created     
        Content-Length: 153     
        Content-Type: application/json; charset=utf-8     
        Location: /v1/queues/samplequeue/messages?ids=51e840e71d10b2055fd565fb,51e840e71d10b2055fd565fc{partial": false, "resources": ["/v1/queues/samplequeue/messages/51e840e71d10b2055fd565fb", "/v1/queues/samplequeue/messages/51e840e71d10b2055fd565fc"]}

Above, if you check the response, you will see that the queue returned
two ids. It is always a good practice to post messages in batches as
network latency will be a smaller factor in overall performance compared
to sending one message at a time.

### Claiming messages

Claiming a message is pretty much like marking a message so it will be
invisible when another worker wants to claim messages. By default 10
messages are claimed. In the sample request below, we will get 2
messages claimed as we use pass 2 as limit.

***Request***

    $ curl -i -X POST https://\
      $ENDPOINT:443/v1/queues/samplequeue/claims?limit=2 -d ' { "ttl": 60, "grace": 60} '\ 
      -H "X-Auth-Token: $TOKEN" \
      -H "Content-type: application/json" -H "Client-ID: e58668fc-26eb-11e3-8270-5b3128d43830"

***Response***

    HTTP/1.1 200 OK Content-Length: 306 
    Content-Type: application/json; charset=utf-8 
    Location: /v1/queues/samplequeue/claims/51e852d01d10b2056dd5703c [{"body": {"event": "two"}, "age": 5, \        "href":"/v1/queues/samplequeue/messages/51e852cb1d10b20571d56f10?claim\_id=51e852d01d10b2056dd5703c",
    "ttl": 300}, {"body": {"event": "three"}, "age": 5, "href":
    "/v1/queues/samplequeue/messages/51e852cb1d10b20571d56f11?claim\_id=51e852d01d10b2056dd5703c",
    "ttl": 120}\]

### Deleting completed messages

***Request***

    $ curl -i -X DELETE https://\
      $ENDPOINT:443/v1/queues/samplequeue/messages/51e852cb1d10b20571d56f10?claim\
      _id=51e852d01d10b2056dd5703c \
      -H "X-Auth-Token: $TOKEN" \
      -H "Content-type: application/json" 
      -H "Client-ID: e58668fc-26eb-11e3-8270-5b3128d43830"

***Response***

    HTTP/1.1 204 No Content

204 is a valid response which validates that there isn't a message with
the given message and claim id. It doesn't necessarily say that message
is deleted, though.

**Note**: 
For additional API documentations, see the following resources: 
- [Rackspace Cloud Queues Getting Started Guide](https://developer.rackspace.com/docs/cloud-queues/v1/developer-guide/#getting-started)
- [Rackspace Cloud Queues API Developer Guide](https://developer.rackspace.com/docs/cloud-queues/v1/developer-guide/#developer-guide)

