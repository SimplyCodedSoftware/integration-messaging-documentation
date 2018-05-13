[Return to table of contents](../index.md)

II. Core Messaging
=================

* [2.4 Transformer](#24-transformer)
    * [2.4.1 Configuring Transformer](#241-configuring-transformer)

2.4 Transformer
============

Message Transformers play a very important role in enabling the loose-coupling of Message Producers and Message Consumers.   
Rather than requiring every Message-producing component to know what type is expected by the next consumer, Transformers can be added between those components. Generic transformers, such as one that converts a String to an XML Document, are also highly reusable.  

2.4.1 Configuring Transformer
-----------

#### [Reference](../definitions.md/#reference) Transformer
 
````php
    TransformerBuilder::create("inputChannelName", "referenceName", 'decodeXml')
        ->withOutputMessageChannel("outputChannelName")
````

Reference Transformer can also be configured via Annotations


````php
    /**
     * @MessageEndpointAnnotation()
     */
    class TransformerWithMethodParameterExample
    {
    
        /**
         * @TransformerAnnotation(inputChannelName="inputChannelName", outputChannelName="outputChannelName")
         */
        public function decodeXml(string $content) : Order
        {
            // decodes to object
        
            return $order;
        }
        
    }
````

When using a POPO, the method that is used for transformation may expect either the Message type or the payload type of inbound Messages.   
It may also accept Message header values either individually or as a full map by using message converters annotations respectively.  
The return value of the method can be any type. If the return value is `itself a Message`, that will be passed along to the transformer’s output channel.

If transformer returns array and payload isn't array, then Integration Messaging assumes it should change headers instead of payload. 
If you need to change payload directly you should use of [Service Activator](../concepts.md/#136-service-activator)   

#### Expression Transformer

Expression transformer, can make use of Symfony Expression Language, to change the message. 

````php
    TransformerBuilder::createWithExpression("inputChannelName", "payload + 3") : self
````

If payload contains 5 in above example it will result in new Message containing 8 as payload.

#### Header Enricher Transformer

Header enricher will transform message's headers by adding ones passed during construction `$messageHeaders`.

````php
    public static function createHeaderEnricher(string $inputChannelName, array $messageHeaders) : self
````

[Jump to 2.5 Message Endpoints...](./message-endpoints.md)
============