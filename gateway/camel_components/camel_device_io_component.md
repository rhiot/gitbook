# Camel Device I/O component

Camel Device I/O component provides a wrapper around a GPIO service feature via [Device I/O](http://openjdk.java.net/projects/dio/) API. Camel Device I/O GPIO component is used to read and write state of the GPIO pins. In future release, it could manage i2c device for example.

## Maven dependency

In order to take advantage of this component, Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-device-io</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

**Available since `rhiot.version` >= 0.1.3**


## URI format for GPIO

    deviceio-gpio://<gpioId>[?options]

Where *gpioId* is the Id of the pin. For example to work with the PIN number 9, use the following URI:

    deviceio-gpio://9


### Optional URI Parameters

| Parameter      | Default value      | Description          |
|----------------|--------------------|----------------------|
| `gpioId`       |                    |                               |
| `state`        |    `false`         | Initial state of the PIN.     |
| `action`       |   `TOGGLE`            | Default : use Body if Action for output Pin (TOGGLE, BLINK, HIGH, LOW)|
| `direction`        |    `DIR_OUTPUT_ONLY` for Producer    `DIR_INPUT_ONLY` for Consumer      | Direction of the PIN.     |
| `mode`        |    `MODE_OUTPUT_PUSH_PULL` for Producer    `MODE_INPUT_PULL_UP` for Consumer      | Mode of the PIN.     |
| `trigger`        |    `TRIGGER_NONE` for Producer,   `TRIGGER_BOTH_EDGES` for Consumer      | Trigger of the PIN.   |
| `shutdownState`        | `false`                       | To configure the pin state value before camel context shutdown        |
| `delay`        | `0`                       | To configure BLINK delay        |
| `duration`     | `50`                      | To configure BLINK duration        |


### Consuming

    from("deviceio-gpio://13")
    .to("log:default?showHeaders=true");

### Producing

    from("timer:default?period=2000")
    .to("deviceio-gpio://4");

When using producer you can also set or override action using message header with a key of `DeviceIOConstants.CAMEL_DEVICE_IO_ACTION`

    from("timer:default?period=2000")
      .setHeader(DeviceIOConstants.CAMEL_DEVICE_IO_ACTION).constant("LOW")
      .to("deviceio-gpio://4?action=TOGGLE");

### Example: Simple button with LED mode

Plug an button on GPIO 1, and LED on GPIO 2 (with Resistor) and code a route like this

    from("deviceio-gpio://1").id("switch-led")
      .to("deviceio-gpio://2");

## URI format for i2c

**Available since `rhiot.version` >= 0.1.5**


    deviceio-i2c://<busId>/<deviceId>[?options]

Where *busId* is the Id of the i2c Bus. Device Id is to locate i2c device on the bus, it depends of driver and/or component address :

    deviceio-i2c://1/0x77
    
#### Tips
   
For RaspberryPi, the bus id should be `1`. To find device address and device plugged on your Raspberry Pi you can use following command :
   
     sudo i2cdetect -y 1


### Optional URI Parameters

| Parameter      | Default value      | Description          |
|----------------|--------------------|----------------------|
| `driver`       |                    | Driver class to load via `META-INF/services/io/rhiot/component/deviceio/i2c/<driver>` file |


The `driver` parameter must match `META-INF/services/io/rhiot/component/deviceio/i2c/<driver>` file name. The `driver` provides correct driver class name to instanciate. A driver must extend `io.rhiot.component.deviceio.i2c.driver.I2CDriverAbstract` class and implements `io.rhiot.component.deviceio.i2c.driver.I2CDriver` interface.

#### i2c driver

| Driver | Feature |
|--------|---------|
| `bmp180`    | [Temperature and Pressure sensor](http://www.adafruit.com/products/1603) |
| `hts221`    | Humidity and Temperature sensor used by the [Official RaspberryPi Sense-Hat](https://www.raspberrypi.org/products/sense-hat/) [ST device doc](http://www.st.com/web/en/resource/technical/document/datasheet/DM00116291.pdf)    |

##### bmp180 driver

Driver Parameter, just add these parameters at the end of the URI

`mode` = `ULTRA_LOW_POWER`,`STANDARD`,`HIGH_RES`,`ULTRA_HIGH_RES`

##### hts221 driver

Driver Parameter, just add these parameters at the end of the URI
  
`bdu` = `BDU_UPDATE_AFTER_READING`, `BDU_CONTINUOUS`

`odr` = `ODR_ONE_SHOT`,`ODR_1_HZ`,`ODR_7_HZ`,`ODR_12DOT5_HZ`

