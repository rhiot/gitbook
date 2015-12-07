# Camel Kura GPIO component

Camel Kura GPIO component provides a wrapper around a GPIO service feature into Kura Platform.
Under the hood, the component uses [Device I/O](http://openjdk.java.net/projects/dio/) library. Camel Kura GPIO 
component can be used to read and write state of the GPIO pins.

## Maven dependency

In order to take advantage of this component, Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-kura</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

Available since Rhiot 0.1.3


## URI format for GPIO

    kura-gpio:gpioId[?options]

Where *gpioId* is a number of the pin. For example to work with the PIN number 9, use the following URI:

    kura-gpio:9


### Optional URI Parameters

| Parameter      | Default value      | Description          |
|----------------|--------------------|----------------------|
| `gpioId`       |                    |                               |
| `state`        |    `false`         | Initial state of the PIN.     |
| `direction`    | `OUTPUT`            | To configure GPIO pin direction, check Kura GPIO library for more details  [KuraGPIODirection](http://download.eclipse.org/kura/releases/1.3.0/docs/apidocs/org/eclipse/kura/gpio/KuraGPIODirection.html)                   |
|`trigger`|`BOTH_EDGES`|To configure GPIO pin trigger, check Kura GPIO library for more details  [KuraGPIOTrigger](http://download.eclipse.org/kura/releases/1.3.0/docs/apidocs/org/eclipse/kura/gpio/KuraGPIOTrigger.html)|
|`mode`|`OUTPUT_PUSH_PULL`|To configure GPIO pin mode, check Kura GPIO library for more details  [KuraGPIOMode](http://download.eclipse.org/kura/releases/1.3.0/docs/apidocs/org/eclipse/kura/gpio/KuraGPIOMode.html)|
| `action`       |               | Default : use Body if Action for output Pin (TOGGLE, BUZZ, HIGH, LOW for digital only)|
| `shutdownState`        | `false`                       | To configure the pin state value before camel context shutdown        |
| `delay`        | `0`                       | To configure BLINK delay        |
| `duration`     | `50`                      | To configure BLINK duration        |


## Consuming:

    from("kura-gpio://13?mode=INPUT&state=false")
    .to("log:default?showHeaders=true");

## Producing

    from("timer:default?period=2000")
    .to("kura-gpio://4?mode=OUTPUT&state=false&action=TOGGLE");

When using producer you can also set or override action using message header with a key of `KuraConstants.CAMEL_KURA_GPIO_ACTION`

    from("timer:default?period=2000")
    .process(exchange -> exchange.getIn().setHeader(KuraGPIOConstants.CAMEL_KURA_GPIO_ACTION, "LOW"))
    .to("kura-gpio://4?mode=OUTPUT&state=false&action=TOGGLE");

## Example: Simple button with LED mode

Plug an button on GPIO 1, and LED on GPIO 2 (with Resistor) and code a route like this

    from("kura-gpio:1?direction=INPUT&state=false").id("switch-led")
      .to("kura-gpio:2?&action=TOGGLE");
