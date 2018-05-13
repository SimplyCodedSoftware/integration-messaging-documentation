[Return to table of contents](../index.md)

II. Core Messaging
=================

2.3 Routing
============

Routers are a crucial element in many messaging architectures. 
They consume Messages from a Message Channel and forward each consumed message to one or more different Message Channel depending on a set of conditions. 

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
  
Points to [Reference](../definitions.md/#reference), that must return target channel name or 
array of channel names. 

````php
    $referenceName = "isHighValueOrder";
    RouterBuilder::create("routingChannelName", $referenceName, "route")
    
    
    
    class isHighValueOrderService
    {
        public function route(Order $order) : string
        {
            if ($order->isHighValueOrder()) {
                return "exlusiveChannelName";
            }
            
            return "normalChannelName";
        }
    }
````

2.3.2 Router configuration
-----------

#### Default Resolution Channel

If router will not receive channel name to resolve, default channel can be set. 

````php
    RouterBuilder::create("routingChannelName", "someReference", "route")
        ->withDefaultResolutionChannel("debugChannel")
````

#### Is Resolution Required

If router will not resolve any channel and default resolution is not set, then it will throw exception on default.  
This behaviour can be changed, so it will behave like [Message Filter](../concepts.md/#132-filter) 


````php
    RouterBuilder::create("routingChannelName", "someReference", "route")
        ->setResolutionRequired(false)
````

[Jump to 2.4 Transformer...](./transformer.md)
============