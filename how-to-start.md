[Return to table of contents](https://simplycodedsoftware.github.io/integration-messaging-documentation)

II. How to start
=================

* [2.1 Installation](#21-installation)
* [2.2 Main Components](#22-main-components)

2.1 Installation
============

2.1.1 Require package
-----------

`Integration Messaging` does not base on any other framework, and thanks to that can be connected to any.  
Currently available implementation is for ready to use for `Symfony`.    
  
```javascript  
"require": {  
    "simplycodedsoftware/integration-messaging-symfony": "^0.4.0"  
}
```

It's highly recommended to use integration messaging with `Symfony` configured with automatic autowire.  
This really simplifiy working with `Integration Messaging`.  


2.1.2 Turn it on 
-----------

If you're using `symfony 3`, activate bundle in your Kernel class.  
`Symfony 4` activate bundle in `config/bundles.php`

````php
    [
        ICTWorks\Matching\Infrastructure\Framework\MatchingBundle::class => ['all' => true]
    ]
````

2.1.3 Annotations 
-----------

Huge part of the high level framework's code is based on `Annotations` to simplify client's API.  
If you having problem loading the annotations or have not registered `AnnotationRegistry`, you need to do it 
in order to work with the framework.  

You should add it to your `console.php` and `app.php`
````php
$loader = require __DIR__.'/../vendor/autoload.php';
\Doctrine\Common\Annotations\AnnotationRegistry::registerLoader(array($loader, 'loadClass'));
````

2.1.4 Quick Start 
-----------

If you want to play around with the concept you may try skeleton project with Symfony 4 and Integration Messaging configured.    
Just clone [skeleton-integration-messaging](https://github.com/SimplyCodedSoftware/skeleton-integration-messaging) and follow `README instructions`