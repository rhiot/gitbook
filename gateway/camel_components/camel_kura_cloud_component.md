# Camel Kura Cloud component

**This component can be used only in the Kura server.**

The Kura platform implements CloudService. CloudService provides some features to push and receive messages. Camel Kura Cloud component interacts directly with CloudService.
Configuration uses the same convention than [CloudService](http://download.eclipse.org/kura/releases/1.3.0/docs/apidocs/org/eclipse/kura/cloud/CloudService.html) and [CloudClient](http://download.eclipse.org/kura/releases/1.3.0/docs/apidocs/org/eclipse/kura/cloud/CloudClient.html) classes from [Kura API](http://download.eclipse.org/kura/releases/1.3.0/docs/apidocs/).

## Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-kura</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

**Avaliable for `rhiot.version` >= 0.1.3**


## URI format for GPIO

    kura-cloud:applicationId/topicId[?options]

both *topicId* *applicationId* must match MQTT topic  pattern.


### Optional URI Parameters

| Parameter        | Default value             | Description                 |
|------------------|---------------------------|-----------------------------|
| `applicationId`          |                           | Kura AppId                  |
| `topic`        |                           | MQTT topicId                |
| `qos`            |0                          | MQTT semantic               |
| `retain`         |false                      |                             |
| `priority`       |5                          | Kura Semantic                           |
| `includeDeviceId`|                           | will pick up [deviceId](http://download.eclipse.org/kura/releases/1.3.0/docs/apidocs/org/eclipse/kura/system/SystemService.html#getSerialNumber%28%29) from Kura API, only works when `control=true` |
| `control`        | false                          |  if true, push a control message, else common message                  |


### Optional Headers

Several URI parameters can me overwrite by the "in Message" headers.

| URI Parameter    | Message Header                 | 
|------------------|--------------------------------|
| `topicId`        |`CamelKuraCloud.topicId`        |
| `qos`            |`CamelKuraCloud.qos`            |
| `retain`         |`CamelKuraCloud.retain`         |
| `priority`       |`CamelKuraCloud.priority`       |
| `includeDeviceId`|`CamelKuraCloud.includeDeviceId`|
| `control`        |`CamelKuraCloud.control`        |

## Consuming:

    from("kura-cloud:app/topic").to("log:result");

## Producing

    from("timer://heartbeat").setBody(constant("Hello"))
        .to("kura-cloud:appId/appTopic?qos=1&retain=false&priority=5"); 
