[Return to table of contents](https://simplycodedsoftware.github.io/integration-messaging-documentation)

II. Core Messaging
=================

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
    
