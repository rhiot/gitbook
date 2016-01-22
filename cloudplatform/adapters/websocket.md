# Websocket protocol adapter

Websocket protocol adapter bridges Websocket requests with the ActiveMQ Transport. It therefore allows you to expose your services via Websocket API.


## Protocol binding rules

Websocket protocol adapter uses STOMP protocol to interact with AMQP queue destination:

Here is a STOMP frame over Websocket 

      SEND
      destination:document.save.stompDoc
      content-type:application/json

      {"foo":"bar"}^@

will map

      amqp:queue:document.save.stompDoc

[More information about STOMP over Websocket](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/websocket.html#websocket-stomp
). 
[You can use your browser](https://github.com/jmesnil/stomp-websocket) to pub/sub some message to CloudPlatform enviroment 

## Starting Websocket protocol adapter

This section describes how to start Websocket protocol adapter.

### Starting Websocket protocol adapter in a PaaS environment

PaaS distribution of Cloud Platform has Websocket protocol adapter included by default (listening on port 9090).

### Starting Websocket protocol adapter programatically in Spring runtime

In the opposite way of the other protocol adapters, Websocket protocol adapter comes from ActiveMQ broker, just need the following jar into your POM file and set several system wide parameters

    	<dependency>
    		<groupId>io.rhiot</groupId>
    		<artifactId>rhiot-cloudplatform-runtime-spring</artifactId>
    		<version>${rhiot.version}</version>
    	</dependency>

Then connect to the Cloud Platform:

        import io.rhiot.cloudplatform.runtime.spring.CloudPlatform
        ...
        System.setProperty("spring.activemq.broker.websocketEnabled", "true");
        System.setProperty("spring.activemq.broker.websocketPort", "9090" );
        new CloudPlatform().start();


## Spring runtime Configuration

By default Websocket connector listens on HTTP port `9090`. You can change this port by setting `spring.activemq.broker.websocketPort` property.