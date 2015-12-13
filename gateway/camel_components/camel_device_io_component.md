# Camel Device I/O component

Camel Device I/O component provides a wrapper around a GPIO service feature via [Device I/O](http://openjdk.java.net/projects/dio/) API. Camel Device I/O component can be used to read and write state of the GPIO pins. In future release, it could manage i2c device for example.

## Maven dependency

In order to take advantage of this component, Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-device-io</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

**Available since Rhiot 0.1.3**


## URI format for GPIO

    deviceio-gpio://<gpioId>[?options]

Where *gpioId* is a number of the pin. For example to work with the PIN number 9, use the following URI:

    deviceio-gpio:9


### Optional URI Parameters

| Parameter      | Default value      | Description          |
|----------------|--------------------|----------------------|
| `gpioId`       |                    |                               |
| `state`        |    `false`         | Initial state of the PIN.     |
| `action`       |               | Default : use Body if Action for output Pin (TOGGLE, BLINK, HIGH, LOW)|
| `direction`        |    `DIR_OUTPUT_ONLY` for Producer    `DIR_INPUT_ONLY` for Consumer      | Direction of the PIN.     |
| `mode`        |    `MODE_OUTPUT_PUSH_PULL` for Producer    `MODE_INPUT_PULL_UP` for Consumer      | Mode of the PIN.     |
| `trigger`        |    `TRIGGER_NONE` for Producer,   `TRIGGER_RISING_EDGE` or(1) `TRIGGER_FALLING_EDGE` for Consumer      | Trigger of the PIN.  (1) [OR Bitwise](https://en.wikipedia.org/wiki/Bitwise_operation#OR)  |
| `shutdownState`        | `false`                       | To configure the pin state value before camel context shutdown        |
| `delay`        | `0`                       | To configure BLINK delay        |
| `duration`     | `50`                      | To configure BLINK duration        |




## Consuming

    from("deviceio-gpio://13")
    .to("log:default?showHeaders=true");

## Producing

    from("timer:default?period=2000")
    .to("deviceio-gpio://4");

When using producer you can also set or override action using message header with a key of `DeviceIOConstants.CAMEL_DEVICE_IO_ACTION`

    from("timer:default?period=2000")
      .setHeader(DeviceIOConstants.CAMEL_DEVICE_IO_ACTION).constant("LOW")
      .to("deviceio-gpio://4?action=TOGGLE");

## Example: Simple button with LED mode

Plug an button on GPIO 1, and LED on GPIO 2 (with Resistor) and code a route like this

    from("deviceio-gpio://1?direction=INPUT&state=false").id("switch-led")
      .to("deviceio-gpio://2?&action=TOGGLE");
