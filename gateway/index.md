# Gateway

*Rhiot gateway* is the small fat jar application that can be installed into the field device. Gateway acts as a bridge
between the sensors and the data center. Under the hood, Rhiot gateway is the fat jar running
[Vert.x](http://vertx.io) and Apache Camel.

Rhiot also supports [Eclipse Kura](https://www.eclipse.org/kura) based gateways. In particular Rhiot gateway can
communicate with Kura gateway (and reversely) deployed on the same device.