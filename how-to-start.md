[Return to table of contents](https://simplycodedsoftware.github.io/integration-messaging-documentation)

II How to start
=================

* [2.1 Installation](#21-installation)
* [2.2 Main Components](#22-main-components)

2.1 Installation
============

`Integration Messaging` does not base on any other framework, and thanks to that can be connected to any.  
Currently available implementation is for ready to use for `Symfony`.    
  
```php  
"require": {  
    "simplycodedsoftware/integration-messaging-symfony": "^0.4.0"  
}
```

It's highly 

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