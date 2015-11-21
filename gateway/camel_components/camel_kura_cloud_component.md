# Camel Kura Cloud component

The Kura platform implements CloudService. CloudService provides some features to push and receive messages from outside. Camel Kura Cloud component interacts directly with CloudService.
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

    pi4j-gpio://gpioId[?options]

*gpioId* must match [A-Z_0-9]+ pattern.
By default, pi4j-gpio uses *RaspiPin* Class, change it via *gpioClass* property
You can use static field name "*GPIO_XX*", pin name "*GPIO [0-9]*" or pin address "*[0-9]*"