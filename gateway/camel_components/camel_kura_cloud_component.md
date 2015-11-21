# Camel Kura Cloud component

The Kura platform implements CloudService. CloudService provides some features to push and receive messages. Camel Kura Cloud component interacts directly with CloudService.
Configuration uses the same convention than [CloudService](http://download.eclipse.org/kura/releases/1.3.0/docs/apidocs/org/eclipse/kura/cloud/CloudService.html) and [CloudClient](http://download.eclipse.org/kura/releases/1.3.0/docs/apidocs/org/eclipse/kura/cloud/CloudClient.html) classes from [Kura API](http://download.eclipse.org/kura/releases/1.3.0/docs/apidocs/).

## Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-kura</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

 Avaliable for rhiot.version >= 0.1.3


## URI format for GPIO

    kura-cloud:appId/topicId[?options]

both *topicId* *appId* must match MQTT topic  pattern.


### Optional URI Parameters

| Parameter        | Default value             | Description                 |
|------------------|---------------------------|-----------------------------|
| `appId`          |                           | Kura AppId                  |
| `topicId`        |                           | MQTT topicId                |
| `qos`            |0                          |                             |
| `retain`         |false                      |                             |
| `priority`       |5                          |                             |
| `includeDeviceId`|                           |                             |


### Optional Headers


