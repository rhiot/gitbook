# Camel Kura Cloud component

The Kura platform comes with *CloudService* which provides opinionated way to push and receive telemetry payloads. Camel
Kura Cloud component provides a wrapper around `CloudService` instance provided by Kura. Kura Cloud component can be
used to bride Camel routes with Kura `CloudService`. In particular to consume messages from
`CloudService` to Camel routes. And to send messages from Camel routes to `CloudService`.

## Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-kura</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

**Available starting from Rhiot 0.1.3**

## URI format

    kura-cloud:applicationId/topicId[?options]

Both *topicId* *applicationId* must match MQTT topic pattern.

Please note that `KuraCloudComponent` comes with a static cache of cloud clients. It means that cloud client for a given
application ID is created only once per Kura server and shared between OSGi bundles.


### Optional URI Parameters

| Parameter        | Default value             | Description                 |
|------------------|---------------------------|-----------------------------|
| `applicationId`          |                           | Kura application ID which can be used to override value from URI. |
| `topic`        |                           | MQTT topicId                |
| `qos`            |0                          | MQTT semantic               |
| `retain`         |false                      |                             |
| `priority`       |5                          | Kura Semantic                           |
| `deviceId`|                           | only works when `control=true` |
| `control`        | false                          |  if true, push a control message, else common message                  |

### Optional Headers

Majority of the producer endpoint options can be overridden by the headers provided in the exchange.

| URI Parameter    | Message Header                 | 
|------------------|--------------------------------|
| `topicId`        |`CamelKuraCloud.topicId`        |
| `qos`            |`CamelKuraCloud.qos`            |
| `retain`         |`CamelKuraCloud.retain`         |
| `priority`       |`CamelKuraCloud.priority`       |
| `deviceId`       |`CamelKuraCloud.deviceId`|
| `control`        |`CamelKuraCloud.control`        |

## Usage

In order to read messages from a cloud service, the following route syntax can be used:

    from("kura-cloud:myApplication/myTopic").
      to("log:result");

In order to send a message to a cloud service, use the producer syntax:

    from("timer://heartbeat").
      setBody(constant("Hello")).
      to("kura-cloud:myApplication/myTopic");

### Configuring endpoints

Cloud Service endpoints can be configured as any Camel component. So for example to apply custom quality of service to
the message sent, use the `qos` endpoint option:

    from("timer://heartbeat").
      setBody(constant("Hello")).
      to("kura-cloud:myApplication/myTopic?qos=2");
