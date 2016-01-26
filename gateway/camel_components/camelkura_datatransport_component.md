# Camel Kura DataTransport component

DataTransportService implementations provide the ability of connecting to a remote broker, publish messages, subscribe to topics, receive messages on the subscribed topics, and disconnect from the remote message broker.
**[from Eclipse Kura JavaDoc](https://github.com/eclipse/kura/blob/develop/kura/org.eclipse.kura.api/src/main/java/org/eclipse/kura/data/DataTransportService.java)** 

## Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-kura</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

**Avaliable for `rhiot.version` >= 0.1.4**


## URI format for Cloud

    kura-datatransport:topic[?options]

Both *topic* must match MQTT topic  pattern.