[Return to table of contents](https://simplycodedsoftware.github.io/integration-messaging-documentation)

II. Core Messaging
=================

2.3 Routing
============

Routers are a crucial element in many messaging architectures. 
They consume Messages from a Message Channel and forward each consumed message to one or more different Message Channel depending on a set of conditions.

Router does return channel name or names, that message will be routed too. 

2.3.1 Router types
-----------

There are few types of routers available out of the box:


#### Payload Type Router

Route the message based on class type of the message payload.

````php
    RouterBuilder::createPayloadTypeRouter(
        "routingChannelName", 
        [
            \stdClass::class => 'channel1',
            Order::class => 'channel2'
        ])
````

#### Header Value Router

A HeaderValueRouter will send Messages to the channel based on the individual header value mappings. When a HeaderValueRouter is created it is initialized with the name of the header to be evaluated. 

````php
    $headerToEvaluateName = "emailType";
    RouterBuilder::createHeaderValueRouter("routingChannelName", $headerName, [
                'private' => 'channel1',
                'public' => 'channel2'
            ])
````

#### Reference Router
  
[Reference](../definitions.md/#reference)

````php
    $referenceName = "isVipOrder";
    RouterBuilder::create("routingChannelName", $referenceName, "route")
````