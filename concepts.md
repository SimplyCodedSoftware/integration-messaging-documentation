Concepts
=================

* [1.1 Background](#11-background)
* [1.2 Main Components](#12-main-components)
    * [1.2.1 Message](#121-message)
    * [1.2.2 Message Channel](#122-message-channel)
    * [1.2.3 Message Endpoint](#123-message-endpoint)
* [1.3 Message Endpoints](#13-message-endpoints)
    * [1.3.1 Transformer](#131-transformer)
    * [1.3.2 Transformer](#132-filter)
    * [1.3.3 Transformer](#133-router)
    * [1.3.4 Transformer](#134-splitter)
    * [1.3.5 Transformer](#135-aggregator)            
    * [1.3.6 Service Activator](#136-service-activator)            

1.1 Background
============

`Integration Messaging` brings programming model based on messaging. It supports message-driven architectures where inversion of control applies to runtime concerns, such as when certain business logic should executed and where the response should be sent. It supports routing and transformation of messages so components can stay loosely coupled for modularity and testability.  
Messaging and integration concerns are handled by the framework, so business components are separated from integration logic.

Integration Messaging design is inspired by [Enterprise Integration Patterns](http://www.enterpriseintegrationpatterns.com/) and [Spring Integration](https://projects.spring.io/spring-integration/). 
Developers who have read the book or have expierence with `Spring Integration Framework` should be immediately comfortable with the concepts and terminology.  


1.2 Main Components
============

Messaging systems typically follow `pipes-and-filters` model. The "filters" represent any component that is capable of `producing` and/or `consuming` messages, and the "pipes" transport the messages between filters so that the components themselves remain loosely-coupled.

1.2.1 Message
-----------
![](https://docs.spring.io/spring-integration/reference/html/images/message.jpg)

A `Message` is a generic wrapper for any PHP object/scalar combined with metadata.  
It consists of a payload and headers. The payload can be of any type and the headers hold commonly required information such as id, timestamp, correlation id, and return address. Headers are also used for passing values to and from connected transports. For example, when creating a Message from a received File, the file name may be stored in a header to be accessed by downstream components. Developers can also store any arbitrary key-value pairs in the headers.

1.2.2 Message Channel
-----------
![](https://docs.spring.io/spring-integration/reference/html/images/channel.jpg)

A Message Channel represents the "pipe" of a pipes-and-filters architecture. Producers send Messages to a channel, and consumers receive Messages from a channel. The Message Channel therefore decouples the messaging components, and also provides a convenient point for interception and monitoring of Messages.

A Message Channel may follow either Point-to-Point or Publish/Subscribe semantics. With a Point-to-Point channel, at most one consumer can receive each Message sent to the channel. Publish/Subscribe channels, on the other hand, will attempt to broadcast each Message to all of its subscribers.  

Whereas "Point-to-Point" and "Publish/Subscribe" define the two options for how many consumers will ultimately receive each Message, there is another important consideration: should the channel buffer messages? Pollable Channels are capable of buffering Messages within a queue. The advantage of buffering is that it allows for throttling the inbound Messages and thereby prevents overloading a consumer. However, as the name suggests, this also adds some complexity, since a consumer can only receive the Messages from such a channel if a poller is configured. On the other hand, a consumer connected to a Subscribable Channel is simply Message-driven.

1.2.3 Message Endpoint
-----------
One of the primary goals of Integration Messaging is to simplify the development of enterprise integration solutions through inversion of control. This means that you should not have to implement consumers and producers directly, and you should not even have to build Messages and invoke send or receive operations on a Message Channel. Instead, you should be able to focus on your specific domain model with an implementation based on plain Objects. Then, by providing configuration, you can "connect" your domain-specific code to the messaging infrastructure provided by Spring Integration. The components responsible for these connections are Message Endpoints. This does not mean that you will necessarily connect your existing application code directly. Any real-world enterprise integration solution will require some amount of code focused upon integration concerns such as routing and transformation. The important thing is to achieve separation of concerns between such integration logic and business logic.  
The next section will provide an overview of the Message Endpoint types that handle these responsibilities, and in upcoming chapters, you will see how configuration options provide a non-invasive way to use each of these.

1.3 Message Endpoints
============

A Message Endpoint represents the "filter" of a pipes-and-filters architecture. As mentioned above, the endpoint’s primary role is to connect domain specific code to the messaging framework and to do so in a non-invasive manner. In other words, the application code should ideally have no awareness of the Message objects or the Message Channels. This is similar to the role of a Controller in the MVC paradigm. Just as a Controller handles HTTP requests, the Message Endpoint handles Messages. Just as Controllers are mapped to URL patterns, Message Endpoints are mapped to Message Channels. The goal is the same in both cases: isolate application code from the infrastructure. These concepts are discussed at length along with all of the patterns that follow in the Enterprise Integration Patterns book. Here, we provide only a high-level description of the main endpoint types that are supported. The chapters that follow will elaborate and provide sample code as well as configuration examples.

1.3.1 Transformer
-----------

A Message Transformer is responsible for converting a Message’s content or structure and returning the modified Message. Similarly, a transformer may be used to add, remove, or modify the Message’s header values.

1.3.2 Filter
-----------

A Message Filter determines whether a Message should be passed to an output channel at all. This simply requires a boolean test method that may check for a particular payload content type, a property value, the presence of a header, etc. If the Message is accepted, it is sent to the output channel, but if not it will be dropped (or for a more severe implementation, an Exception could be thrown). Message Filters are often used in conjunction with a Publish Subscribe channel, where multiple consumers may receive the same Message and use the filter to narrow down the set of Messages to be processed based on some criteria
  
     Be careful not to confuse the generic use of "filter" within the Pipes-and-Filters architectural pattern with this specific endpoint type that selectively narrows down the Messages flowing between two channels. The Pipes-and-Filters concept of "filter" matches more closely with Spring Integration’s Message Endpoint: any component that can be connected to Message Channel(s) in order to send and/or receive Messages.
     
1.3.3 Router
-----------
![](https://docs.spring.io/spring-integration/reference/html/images/router.jpg)

A Message Router is responsible for deciding what channel or channels should receive the Message next (if any). Typically the decision is based upon the Message’s content and/or metadata available in the Message Headers. A Message Router is often used as a dynamic alternative to a statically configured output channel on a Service Activator or other endpoint capable of sending reply Messages. 

1.3.4 Splitter
-----------
 
A Splitter is another type of Message Endpoint whose responsibility is to accept a Message from its input channel, split that Message into multiple Messages, and then send each of those to its output channel. This is typically used for dividing a "composite" payload object into a group of Messages containing the sub-divided payloads.
 
1.3.5 Aggregator
-----------
 
 Basically a mirror-image of the Splitter, the Aggregator is a type of Message Endpoint that receives multiple Messages and combines them into a single Message. In fact, Aggregators are often downstream consumers in a pipeline that includes a Splitter. Technically, the Aggregator is more complex than a Splitter, because it is required to maintain state (the Messages to-be-aggregated), to decide when the complete group of Messages is available, and to timeout if necessary. Furthermore, in case of a timeout, the Aggregator needs to know whether to send the partial results or to discard them to a separate channel.
 
     Aggregate implementation is in progress
 
1.3.6 Service Activator
-----------

A Service Activator is a generic endpoint for connecting a service instance to the messaging system. The input Message Channel must be configured, and if the service method to be invoked is capable of returning a value, an output Message Channel may also be provided.

    The output channel is optional, since each Message may also provide its own Return Address header. This same rule applies for all consumer endpoints.

The Service Activator invokes an operation on some service object to process the request Message, extracting the request Message’s payload and converting if necessary (if the method does not expect a Message-typed parameter). Whenever the service object’s method returns a value, that return value will likewise be converted to a reply Message if necessary (if it’s not already a Message). That reply Message is sent to the output channel. If no output channel has been configured, then the reply will be sent to the channel specified in the Message’s "return address" if available.

![](https://docs.spring.io/spring-integration/reference/html/images/handler-endpoint.jpg)

```php
/**
 * @MessageEndpointAnnotation()
 */
class ServiceActivatorWithAllConfigurationDefined
{
    /**
     * @param string    $content
     *
     * @return void
     * @ServiceActivatorAnnotation(inputChannelName="inputChannel")
     */
    public function sendMessage(string $content) : void
    {
        // send message
    }
}
```

##### License Information
Big part of documentation was based on [Spring Integration](https://docs.spring.io/spring-integration/reference/html/overview.html). 
https://github.com/spring-projects/spring-framework/blob/master/src/docs/dist/license.txt