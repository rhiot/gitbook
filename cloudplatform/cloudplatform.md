# Rhiot Cloud Platform

The Internet of Things is all about the communication and messaging. Devices connected to the IoT system have to
connect to a kind of centralized hub that allows them to exchange their data with the other devices and backend
services. The device that can't be properly connected to the rest of the application ecosystem, is useless from the IoT
point of view.

If you are looking for such centralized event hub, Rhiot project provides such central bus in the form of the
*Rhiot Cloud Platform*. Rhiot Cloud Platform is a set of backend *services* providing common IoT-related functionalities.
In order to connect to those services using a protocol of your choice, you can use *protocol adapters*. Messages
are exchanged between protocol adapters and backend services using *IoT Connector*. The IoT Connector is an AMQP-based
cloud-ready messaging infrastructure.

The high-level architecture diagram of the Rhiot Cloud Platform is presented on the image below:

 <img src="rhiot_cloud_platform_arch.png" align="center" height="500">

Rhiot comes with a predefined set of Protocol Adapters and backend services that can be used out of the box with your
IoT solution. Rhiot also provides AMQP IoT Connector implementation based on
[ActiveMQ message broker](http://activemq.apache.org).

It is important to stress that only client sending a payload to the Cloud Platform and service consuming message from
the IoT connector are aware of messages semantics. It basically means that it is client responsibility to properly
encode the message, while it is service responsibility to decode the message. Protocol adapters and IoT connector
are not aware of the message semantics.