[Return to table of contents](../index.md)

II. Core Messaging
=================

2.5 Message Endpoints
============

As mentioned in the overview, Message Endpoints are responsible for connecting the various messaging components to channels. Over the next several chapters, you will see a number of different components that consume Messages. Some of these are also capable of sending reply Messages. Sending Messages is quite straightforward. As shown above in [Message Channels](./message-and-channels.md/#211-message-channel), it’s easy to send a Message to a Message Channel. 
Whenever you create message component it's wrapped with one of the consumer types.
And there are two types of consumers: [Polling Consumers](../definitions.md/#pollable-consumer) and [Event Driven Consumers](../definitions.md/#event-driven-consumer).

2.5.1 Message Handler
-----------

MessageHandler interface is implemented by many of the components within the framework.  
In other words, this is not part of the public API, and a developer would not typically implement MessageHandler directly.  
Nevertheless, it is used by a Message Consumer for actually handling the consumed Messages, and so being aware of this strategy interface does help in terms of understanding the overall role of a consumer.

````php
interface MessageHandler
{
    public function handle(Message $message) : void;
}
````

Despite its simplicity, this provides the foundation for most of the components that will be covered in the following chapters (Routers, Transformers, Splitters, Aggregators, Service Activators, etc).   
Those components each perform very different functionality with the Messages they handle, but the requirements for actually receiving a Message are the same, and the choice between polling and event-driven behavior is also the same.   
There are two endpoint implementations that host these callback-based handlers and allow them to be connected to Message Channels.  


2.5.2 Event Driven Consumer
-----------

You may recall that the [SubscribableChannel](../concepts.md/#122-message-channel) provides a subscribe() method and that the method accepts a MessageHandler parameter

````php
    public function subscribe(MessageHandler $messageHandler) : void;
````

2.5.3 Pollable Consumer
-----------

Is currently in developing progress. As will involve spawning new PHP process to start polling messages from channel.

General idea is to not care about infrastructure details. So the final solution, will provide you with list of  
consumers that are require to be run based on your configuration.   
It will allow for generating configuration during Continues Integration for example.


2.5.4 Messaging Gateways
-----------

The primary purpose of a Gateway is to hide the messaging API provided by Integration Messaging. It allows your application’s business logic to be completely unaware of the Spring Integration API and using a generic Gateway, your code interacts instead with a simple interface, only.
As you need to start messaging flow and values that comes from outside are not messages, you need to convert it. That's the role of Gateway.   

#### Gateway Proxy 

As mentioned above, it would be great to have no dependency on the Integration Messaging API at all - including the gateway class.  
Integration Messaging solution to it, is to generate proxy classes based on interface. 

````php
    /**
    * @MessageEndpointAnnotation()
    */
    interface OrderGateway
    {
        /**
         * @GatewayAnnotation(requestChannel="requestChannel", parameterConverters={
         *      @ParameterToPayloadAnnotation(parameterName="orderId")
         * })
         */
        public function buy(string $orderId) : void;
    }
````
 
Above interface will auto-register in your Dependency Injection Container under interface class name.  
This will allow you to inject it anywhere as normal service (auto-wire will work out of the box).  
If you need to register gateway under specific name you can add `referenceName` to `MessageEndpointAnnotation`

````php
    @MessageEndpointAnnotation(referenceName="someReferenceName")
````   

As you can see there is parameter called `parameterConverters`. This is part responsible conversion of outside parameters to message.    


#### Parameter To Message Converter

There are three types of converters available for Gateway.

    @ParameterToPayloadAnnotation(parameterName="orderId")
    @ParameterToHeaderAnnotation(parameterName="userName", headerName="user")
    @ParameterToStaticHeaderAnnotation(headerName="isAdmin", headerValue=true)

````php
    @ParameterToPayloadAnnotation(parameterName="orderId")
````    

Value that will be passed to parameter under `orderId` will be used as message's payload.

````php
    @ParameterToHeaderAnnotation(parameterName="userName", headerName="user")
````

Value that will be passed to parameter under `userName`, will be add to message under header name `user`

 
````php
    ParameterToStaticHeaderAnnotation(headerName="isAdmin", headerValue=true)
````

Is different that converters above, because do not use parameter. It takes value under `headerValue` key 
and assign it to `headerName`. So in above result will be `isAdmin` header containing true.  


The default message converter, when there is only one parameter is `@ParameterToPayloadAnnotation`

#### Request Channel

After conversion the message need to be send somewhere to begin the flow.  
The message under `requestChannel` will be used as channel to send. 

#### Void Return Type

````php
    /**
    * @MessageEndpointAnnotation()
    */
    interface OrderGateway
    {
        /**
         * @GatewayAnnotation(requestChannel="requestChannel", parameterConverters={
         *      @ParameterToPayloadAnnotation(parameterName="orderId")
         * })
         */
        public function buy(string $orderId) : void;
    }
````

#### With Return Type

If return type if void, then message will goes through configured endpoints like transformers, service activators connected by message channels  
and will not return for reply.

  
 ````php
     /**
     * @MessageEndpointAnnotation()
     */
     interface OrderGateway
     {
         /**
          * @GatewayAnnotation(requestChannel="requestChannel", parameterConverters={
          *      @ParameterToPayloadAnnotation(parameterName="orderId")
          * })
          */
         public function getOrder(string $orderId) : Order;
     }
 ````
 
A Gateway will auto-create a temporary, anonymous reply channel in message headers, where it will listen for the reply.  
This is taken from user, and thanks to that OrderGateway what happens during flow. It just knows, that last messaging component
that will no have outputChannel defined, will use  **MessageHeaders::REPLY_CHANNEL**, which is in fact reply channel defined by gateway.  
  
In the above example payload of reply message will require to be Order, otherwise exception will be thrown.  
If interface doesn't return null by using question mark type hint `?Order`, then exception will be thrown if there will be no reply. 

#### Error Channel

````php
          @GatewayAnnotation(
               errorChannel="errorChannel"
          )
````

If gateway method throws exception it will be rethrown by gateway, if no error channel is defined.   
Otherwise the exception will be converted to [ErrorMessage](./message-and-channels.md/#225-message-implementations)
and send to defined channel. This will result in gateway method returning null, so it has to allow for nullable return values.  

#### Transaction Factories

````php
          @GatewayAnnotation(
               transactionFactories={"transactionService"}
          )
````

As gateway it starting point for the message flow, you may want to start transactions at that time.  
To allow for that you need to register within your container Transaction service implementing `TransactionFactory` and implement `Transaction` for that factory. 

````php
    interface TransactionFactory
    {
        /**
         * Create a new transaction and associate it with current process
         *
         * @return Transaction
         */
        public function begin() : Transaction;
    }
    
    interface Transaction
    {
        // methods to implement
    }
````

