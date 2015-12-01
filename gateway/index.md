# Gateway

Rhiot gateway support is primarily focused on providing some additional utilities to make working with 
[Eclipse Kura](https://www.eclipse.org/kura) based gateways easier.

*Rhiot lightweight gateway* is the small fat jar application that can be installed into the field device. Gateway acts as a bridge
between the sensors and the data center. Under the hood, Rhiot gateway is the fat jar running
[Vert.x](http://vertx.io) and Apache Camel.