# Camel Kura GPIO component

Camel Kura GPIO component can be used to manage GPIO feature into Kura Platform.
When it runs into RaspberryPi, this component uses [Device I/O](http://openjdk.java.net/projects/dio/) lib.

## Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-kura</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

 Avaliable for rhiot.version >= 0.1.3


## URI format for GPIO

    kura-gpio://gpioId[?options]

*gpioId* must match [0-9]+ pattern.

You can use static field name "*GPIO_XX*", pin name "*GPIO [0-9]*" or pin address "*[0-9]*"


### Optional URI Parameters

| Parameter            | Default value             | Description                         |
|----------------------|---------------------------|-------------------------------------|
| `gpioId`       |        |                       |
| `state`        |    `false`    | start state                       |
| `direction`    | `DIGITAL_OUTPUT`            | To configure GPIO pin mode, Check Pi4j library for more details                     |
| `action`       |               | Default : use Body if Action for output Pin (TOGGLE, BUZZ, HIGH, LOW for digital only)|
| `shutdownState`        | `false`                       | To configure the pin state value before camel context shutdown        |


## Consuming:

    from("kura-gpio://13?mode=DIGITAL_INPUT&state=LOW")
    .to("log:default?showHeaders=true");

## Producing

    from("timer:default?period=2000")
    .to("kura-gpio://4?mode=DIGITAL_OUTPUT&state=LOW&action=TOGGLE");

When using producer you can also set or override action using message header with a key of `KuraConstants.CAMEL_RBPI_PIN_ACTION`

    from("timer:default?period=2000")
    .process(exchange -> exchange.getIn().setHeader(Pi4jConstants.CAMEL_RBPI_PIN_ACTION, "LOW"))
    .to("kura-gpio://4?mode=DIGITAL_OUTPUT&state=LOW&action=TOGGLE");

##### Simple button w/ LED mode

Plug an button on GPIO 1, and LED on GPIO 2 (with Resistor) and code a route like this

    from("kura-gpio://1?mode=DIGITAL_INPUT&state=HIGH").id("switch-led")
    .to("kura-gpio://2?&action=TOGGLE");
