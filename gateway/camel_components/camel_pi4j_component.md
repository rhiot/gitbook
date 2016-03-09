# Camel Pi4j component

Camel Pi4j component can be used to manage GPIO and I2C bus features from Raspberry Pi.
This component uses [pi4j](http://pi4j.com) library

## Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-pi4j</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

**Avaliable for `rhiot.version` >= 0.1.1**


## URI format for GPIO

    pi4j-gpio://gpioId[?options]

*gpioId* must match [A-Z_0-9]+ pattern.
By default, pi4j-gpio uses *RaspiPin* Class, change it via *gpioClass* property
You can use static field name "*GPIO_XX*", pin name "*GPIO [0-9]*" or pin address "*[0-9]*"


### Optional URI Parameters

| Parameter            | Default value             | Description                                               |
|----------------------|---------------------------|-----------------------------------------------------------|
| `gpioId`               |                           |                                                           |
| `state`                |                           | Digital Only: if input mode then state trigger event, if output then started value                       |
| `mode`                 | `DIGITAL_OUTPUT`            | To configure GPIO pin mode, Check Pi4j library for more details                     |
| `action`               |                           | Default : use Body if Action for output Pin (TOGGLE, BUZZ, HIGH, LOW for digital only) (HEADER digital and analog) |
| `value`                | `0`                         | Analog or PWN Only                       |
| `shutdownExport`       | `true`                      | To configure the pin shutdown export                      |
| `shutdownResistance`   | `OFF`                       | To configure the pin resistance before exit program                      |
| `shutdownState`        | `LOW`                       | To configure the pin state value before exit program                      |
| `pullResistance`       | `PULL_UP`                   | To configure the input pull resistance, Avoid strange value for info http://en.wikipedia.org/wiki/Pull-up_resistor                     |
| `gpioClass`            | `com.pi4j.io.gpio.RaspiPin` | `class<com.pi4j.io.gpio.Pin>` pin implementation                  |
| `controller`           | `com.pi4j.io.gpio.impl.GpioControllerImpl`            | `instance of <com.pi4j.io.gpio.GpioController>` GPIO controller instance, check gpioClass pin implementation to use the same  |

## Consuming:

    from("pi4j-gpio://13?mode=DIGITAL_INPUT&state=LOW")
    .to("log:default?showHeaders=true");

## Producing

    from("timer:default?period=2000")
    .to("pi4j-gpio://GPIO_04?mode=DIGITAL_OUTPUT&state=LOW&action=TOGGLE");

When using producer you can also set or override action using message header with a key of `Pi4jConstants.CAMEL_RBPI_PIN_ACTION`

    from("timer:default?period=2000")
    .process(exchange -> exchange.getIn().setHeader(Pi4jConstants.CAMEL_RBPI_PIN_ACTION, "LOW"))
    .to("pi4j-gpio://GPIO_04?mode=DIGITAL_OUTPUT&state=LOW&action=TOGGLE");

##### Simple button w/ LED mode

Plug an button on GPIO 1, and LED on GPIO 2 (with Resistor) and code a route like this

    from("pi4j-gpio://1?mode=DIGITAL_INPUT&state=HIGH").id("switch-led")
    .to("pi4j-gpio://2?&action=TOGGLE");


### URI format for I2C

    pi4j-i2c://busId/deviceId[?options]

#### Optional URI Parameters

| Parameter            | Default value             | Description                                               |
|----------------------|---------------------------|-----------------------------------------------------------|
| `busId`              |                           | i2c bus                                                   |
| `deviceId`           |                           | i2c device                                                |
| `address`            |  `0x00`                   | address to read                                           |
| `readAction`         |                           | READ, READ_ADDR, READ_BUFFER, READ_ADDR_BUFFER            |
| `size`               |  `-1`                     |                                                           |
| `offset`             |  `-1`                     |                                                           |
| `bufferSize`         |  `-1`                     |                                                           |
| `driver`             |                           | cf available i2c driver                                   |

I2C component is really simplistic, the consumer endpoint reads a single byte, or to a buffer of bytes directly from the device.
The producer writes 1 or more bytes.

For smarter devices, you must implement a driver.

#### i2c driver

| Driver            | Feature                                                            |
|-------------------|--------------------------------------------------------------------|
| `bmp180`            | Temperature and Pressure sensor   (http://www.adafruit.com/products/1603) |
| `tsl2561`           | Light sensor            (http://www.adafruit.com/products/439)     |
| lsm303-accel      | Accelerometer sensor    (http://www.adafruit.com/products/1120)    |
| lsm303-magne      | Magnetometer sensor     (http://www.adafruit.com/products/1120)    |
| mcp23017-lcd      | LCD 2x16 char           (http://www.adafruit.com/products/1109)    |
| hts221      |  Humidity and Temperature sensor used by the [Official RaspberryPi Sense-Hat](https://www.raspberrypi.org/products/sense-hat/) [ST device doc](http://www.st.com/web/en/resource/technical/document/datasheet/DM00116291.pdf)    |
| lps25h      |  Pressure and Temperature sensor used by the [Official RaspberryPi Sense-Hat](https://www.raspberrypi.org/products/sense-hat/) [ST device doc](http://www.st.com/web/en/resource/technical/document/datasheet/DM00066332.pdf)    |


##### bmp180

##### tsl2561



