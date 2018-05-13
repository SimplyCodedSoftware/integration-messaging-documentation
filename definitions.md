[Return to table of contents](../index.md)

Definitions
=================

Reference
-----------

Integration Messaging comes with concept of `Reference/Reference Name`.   
Reference is string, that point to specific class in Dependency Container.    
Thanks to that messaging components can be created without direct reference to the object and resolve it 
at the specific required moment. 

If you make use of symfony auto-wire system, it does register components under class name. 
So you can register messaging component like this:

````php
    RouterBuilder::create("routingChannelName", IsHighValueOrderService::class, "route")
    
    class IsHighValueOrderService
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

If you register e.g. Service Activator using annotations, you do not need to do any extra work in case of auto-wire.

````php
    /**
     * @MessageEndpointAnnotation()
     */
    class OrderService
    {
        /**
         * @ServiceActivatorAnnotation(inputChannelName="inputChannel")
         */
        public function sendMessage(Order $order) : void
        {
            // do something with order
        }
    }
````

In case if you do not use, you need to tell MessageEndpointAnnotation under which name this class is registered:

````php
    @MessageEndpointAnnotation(referenceName="orderService")
````


Event Driven Consumer
-----------
[Read here](http://www.enterpriseintegrationpatterns.com/patterns/messaging/EventDrivenConsumer.html)

Pollable Consumer
-----------
[Read here](http://www.enterpriseintegrationpatterns.com/patterns/messaging/PollingConsumer.html)

Message Components
-----------

Component are things like Service Activator, Transformer, Router, Splitter, Aggregator etc...