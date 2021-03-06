# Camel Kura CloudService bridge

Camel Cloud Service bridge is a Camel-based implementation of the `org.eclipse.kura.cloud.CloudService` interface
(`io.rhiot.component.kura.camelcloud.DefaultCamelCloudService`) providing Camel-based implementation of the
 `org.eclipse.kura.cloud.CloudClient` interface (`io.rhiot.component.kura.camelcloud.CamelCloudClient`). Camel-based
cloud service allows you to sent `KuraPayload` instances to any endpoint supported by Camel, not only to MQTT brokers.

## Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-kura</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

**Available starting from Rhiot >= 0.1.4**

## Working with CamelKuraService

In order to use Camel `CloudService` create new instance taking `CamelContext` as a parameter:

    CloudService cloudService = new DefaultCamelCloudService(camelContext);
    CloudClient cloudClient = cloudService.newCloudClient("myapp");
    cloudClient.publish("amqp:myqueue", "hello world".getBytes(), 0, true, 5);

As you can see, cloud topic is converted to Camel endpoint ([AMQP endpoint](http://camel.apache.org/amqp) in this
particular example).

To subscribe to a given endpoint, use `CloudService#subscribe` method:

    CloudService cloudService = new DefaultCamelCloudService(camelContext);
    CloudClient cloudClient = cloudService.newCloudClient("myapp");
    cloudClient.subscribe("amqp:myqueue");

All messages consumed from the given endpoint will be enqueued in a ([SEDA queue](http://camel.apache.org/seda), named
`seda:APPLICATION_ID`, where `APPLICATION_ID` matches application ID of the cloud client (`myapp` in the example above).

## Handling MQTT idioms

`CloudService` operates on MQTT idioms (like *retain* or *QOS*) which can't be directly mapped into an arbitrary Camel
endpoint. For those reasons we hold those information in exchange headers, so end user can decide how to interpret those.

For example messages published by Camel `CloudService` carry the following headers:

| Message Header                 | Type |  Description        |
|------------------|----------------------|----------|
| `CamelKuraCloudService.control`  |  *boolean*    | Indicates if message if control one or regular one. |
| `CamelKuraCloudService.messageId`   |  *String*  |  Holds unique message identifier generated by *CloudService* . |
| `CamelKuraCloudService.deviceId`|  *String*  | Identifies target device (only for control messages).) |
| `CamelKuraCloudService.qos`            |  *int*  | Expected *Quality Of Service*. |
| `CamelKuraCloudService.retain`         |  *boolean*  | Indicates if message retain is expected.  |
| `CamelKuraCloudService.priority`       |  *int*  | Indicates expected message priority, where *0* is top priority and *5* is regular application priority. |
