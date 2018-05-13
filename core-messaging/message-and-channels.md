[Return to table of contents](../index.md)

II. Core Messaging
=================

* [2.1 Background](#21-message-channels)
    * [2.1.1 Message Channel](#211-message-channel)
    * [2.1.2 Channel Types](#212-channel-types)
    * [2.1.3 Message Channel Implementation](#213-message-channel-implementations)
* [2.2 Message](#22-message)
    * [2.2.1 Message Interface](#221-message-interface)
    * [2.2.2 Message Headers](#222-message-headers)
    * [2.2.3 Message Mutability](#223-message-mutability)            
    * [2.2.4 Message Headers Propogation](#224-message-headers-propagation)            
    * [2.2.5 Message Implementations](#225-message-implementations)            
    * [2.2.6 Message Builder](#226-message-builder)            

2.1 Message Channels
============

While the Message plays the crucial role of encapsulating data, it is the MessageChannel that decouples message producers from message consumers.

2.1.1 Message Channel
-----------

#### Message Channel Interface

Top Message Channel interface follows

````php
interface MessageChannel
{
    public function send(Message $message) : void;
}
````

#### Pollable Channel Interface

Since Message Channels may or may not `buffer` Messages, there are two sub-interfaces defining the buffering (pollable) and non-buffering (subscribable) channel behavior.

````php
interface PollableChannel extends MessageChannel
{
    public function receive() : ?Message;
}
````

#### Subscribable Channel Interface

````php
interface SubscribableChannel extends MessageChannel
{
    public function subscribe(MessageHandler $messageHandler) : void;

    public function unsubscribe(MessageHandler $messageHandler) : void;
}
````

2.1.2 Channel Types
-----------

#### Way of message delivery

There are to way, that messages are handled. 
`Event-Driven/Message-Driven channels` the messaging component is called by the channel. The channel decides `when` the message is 
going to be received. 
`Pollable channels` the messaging component decides when the want to poll the message out of the channel.  

#### Way of message subscription

`Point-To-Point` the message always goes to one message handler (messaging component). 
`Broadcast` the messages goes to multiple message handlers.

2.1.3 Message Channel Implementations
-----------

There are several implementation available out of the box. 

#### PublishSubscribeChannel (Event-Driven)

Allows to add register multiple Message Handlers, that will be informed in Event Driven way.
There is no poll concept in this one. Message are delivered just after channel receives the message.
This is most used for sending Event Messages, since one channel may be connected with multiple handlers. 

#### Queue Channel

Queue channel is `point to point` `pollable` channel.    
Even if the channel has multiple consumers, only one of them should receive any Message sent to that channel.  
Queue channel buffer messages in memory, so after PHP process will die the messages will be lost.  
If you need to store messages between requests you may use `Amqp Queue` Channel.  
In progress there is concept of `message store`, that will be available to store Queue Channel messages. 

#### Direct Channel

The DirectChannel has point-to-point semantics but otherwise is more similar to the PublishSubscribeChannel than pollable channel.  
It implements the SubscribableChannel interface instead of the PollableChannel interface, so it dispatches Messages directly to a subscriber.  
It differs from the PublishSubscribeChannel in that it will only send each Message to a single subscribed MessageHandler

#### Channel Interceptors 

One of the advantages of a messaging architecture is the ability to provide common behavior and capture meaningful information about the messages passing through the system in a non-invasive way.  
Since the Messages are being sent to and received from MessageChannels, those channels provide an opportunity for intercepting the send and receive operations

````php
interface ChannelInterceptor
{
    public function preSend(?Message $message, MessageChannel $messageChannel) : ?Message;

    public function postSend(?Message $message, MessageChannel $messageChannel, bool $wasSuccessful) : void;

    public function preReceive(MessageChannel $messageChannel) : void;

    public function postReceive(?Message $message, MessageChannel $messageChannel) : void;
}
````

    Keep in mind that receive() calls are only relevant for PollableChannels. 
    Therefore, the preReceive(..), postReceive(..) interceptor methods are only invoked when the interceptor is applied to a PollableChannel.
    
2.2 Message
============

Message is a generic container for data. Any object can be provided as the payload, and each Message also includes headers containing user-extensible properties as key-value pairs.

2.2.1 Message Interface
-----------

````php
interface Message
{
    public function getHeaders() : MessageHeaders;

    public function getPayload();
}    
````

Message is most important concept in `Integration Messaging`. 
It's wrapper for data that is passed between messaging components.  
As it is simple Interface and lack of knowledge of data's type, the application can evolve to support new types  
and messaging system will not be affected by changes.  
On the other hand, when some component in the messaging system does require access to information about the Message, such metadata can typically be stored to and retrieved from the metadata in the Message Headers.  

2.2.2 Message Headers
-----------

````php
    class MessageHeaders
    {
        final public function headers() : array
    
        final public function get(string $headerName)
    }

````

Headers are used to carry meta data about information, so you may extract only information you need without touching the payload.  
They are simple hash map structure.

````php
 $someValue = message->get("someKey");
````

There are two predefined headers, that are always available within `Message`

````php
    MessageHeaders::MESSAGE_ID // Type of Uuid
    MessageHeaders::TIMESTAMP // Type of int 
````

Message Id and Timestamp are always generated during message creation / mutation. 
Whenever message is changed new Id and timestamp will be generated. 

2.2.3 Message Mutability
-----------

Message itself is Value Object, which means should not be changed without creating new one.   
Message headers can't be ever changed and do not even expose method for state mutation.  
Message payload is in fact left to the user, as payload may be simple `POPO` with `Setter methods`, that can be modified by reference
from outside.  
General advice it to keep message `immutable`, as this may leads to undesired behaviour. 

2.2.4 Message Headers Propagation
-----------

When messages are processed (and modified) by message-producing endpoints (such as a service activator), in general, inbound headers are propagated to the outbound message.   
One exception to this is a transformer, when a complete message is returned to the framework; in that case, the user code is responsible for the entire outbound message.   
When a transformer just returns the payload; the inbound headers are propagated. Also, a header is only propagated if it does not already exist in the outbound message, allowing user code to change header values as needed.


2.2.5 Message Implementations
-----------

#### Generic Message
Is base message implementation, that is mostly used by the framework to create message.

````php
    GenericMessage::create("payload", MessageHeaders::createEmpty());
````

#### Error Message
Error message is created when error occurs in messaging system and is configured to publish Exceptions to some channel.  
Instead of exception going straight to the top, resulting in User Exception it will be catch and published to specific channel.
You will mostly not need to create those, as this is taken from you.  

````php
    public static function createWithOriginalMessage(\Throwable $exception, Message $originalMessage) : self
````

2.2.6 Message Builder 
-----------

You may notice that the Message interface defines retrieval methods for its payload and headers but no setters. The reason for this is that a Message cannot be modified after its initial creation.
When a Message instance is sent to multiple consumers one of those consumers needs to send a reply with a different payload type, it will need to create a new Message. As a result, the other consumers are not affected by those changes.
Rather than requiring the creation and population of a Map to pass into the GenericMessage factory method, Integration Messaging does provide a far more convenient way to construct Messages: `MessageBuilder`

````php
    final class MessageBuilder
    {
        public static function fromMessage(Message $message) : self
        
        public function setHeader(string $headerName, $headerValue) : self
        
        public static function withPayload($payload) : self
        
        public function build() : Message
    }
````

````php
        $messageBuilder = MessageBuilder::fromMessage($message);
        $messageBuilder->setPayload("newPayload");
        
        $message = $messageBuilder->build();
````

[Jump to 2.3 Routing...](./routing.md)
============