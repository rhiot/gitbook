# Rhiot IoT components

Rhiot project brings some extra components for the Apache Camel intended to make both device-side and server-side IoT
development easier.

## Camel Bluetooth component

Camel Bluetooth component can be used to retrieve the information about the bluetooth devices available within the device
range. Under the hood Bluetooth component uses bluecove library (http://www.bluecove.org/). Bluetooth component supports
both the consumer and producer endpoints.

### Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-bluetooth</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

 Avaliable for rhiot.version >= 0.1.3

### URI format

    bluetooth:label

For example to find all bluetooth devices available near the device, the following route can be used:

    from("bluetooth:scan").to("mock:devices");

The message body is a `io.rhiot.component.bluetooth.BluetoothDevice` instance:

    BluetoothDevices[] devices = consumerTemplate.receiveBody("bluetooth:scan", BluetoothDevices[].class);

You can also request the bluetooth device scanning using the producer endpoint:

    from("direct:scan").to("bluetooth:scan").to("mock:devices");

Or using the producer template directly:

    BluetoothDevices[] devices = template.requestBody("bluetooth:scan", BluetoothDevices[].class);

#### Options

| Option                   | Default value                                                                 | Description   |
|:-------------------------|:-----------------------------------------------------------------------       |:------------- |
| `consumer.initialDelay`  | 1000                                                                          | Milliseconds before the polling starts. |
| `consumer.delay`         | 5000 | Delay between each bluetooth device scan. |
| `consumer.useFixedDelay` | false | Set to true to use a fixed delay between polls, otherwise fixed rate is used. See ScheduledExecutorService in JDK for details. |
| `bluetoothDevicesProvider`   | `new BluecoveBluetoothDeviceProvider()`                                               | reference to the`io.rhiot.component.bluetooth.BluetoothDevicesProvider` instance used to scan bluetooth devices. |


#### Installer

The Bluetooth component installs it's own dependencies for Debian-based systems using apt-get, these include blueman and libbluetooth-dev,
as well as their dependencies.
You can configure the installer or set an alternate one on the component:

    BluetoothComponent bluetooth = new BluetoothComponent();
    bluetooth.setInstaller(new CustomInstaller());
    camelContext.addComponent("bluetooth", bluetooth);

You can also specify alternate/additional dependencies for your platform, if your platform uses my-custom-tools for example, you should configure the component as follows:

    BluetoothComponent bluetooth = new BluetoothComponent();
    bluetooth.setRequiredPackages("my-custom-tools,blueman,libbluetooth-dev"); //comma-separated list of packages to install
    camelContext.addComponent("bluetooth", bluetooth);

If an Installer is not set on the component, Camel will try to find an instance in the
[registry](http://camel.apache.org/registry.html). So for example for Spring application, you can configure the installer as a bean:

    @Bean
    Installer myInstaller() {
        new CustomInstaller();
    }

### Camel GPS BU353 component

**Deprecated:** BU353 component is deprecated and will be removed soon on the behalf of the
[GPSD component](https://github.com/rhiot/rhiot/blob/master/docs/readme.md#camel-gpsd-component).

[BU353](http://usglobalsat.com/p-688-bu-353-s4.aspx#images/product/large/688_2.jpg) is one of the most popular and cheapest GPS units on the market.
It is connected to the device via the USB port. If you are looking for good and cheap GPS receiver for your IoT solution, definitely consider purchasing this unit.

Camel GPS BU353 component can be used to read current GPS information from that device. With Camel GPS BU353 you can
just connect the receiver to your computer's USB port and read the GPS data - the component
will make sure that GPS daemon is up, running and
switched to the [NMEA mode](http://www.gpsinformation.org/dale/nmea.htm). The component also takes care of parsing the
NMEA data read from the serial port, so you can enjoy the `io.rhiot.component.gps.bu353.ClientGpsCoordinates`
instances received by your Camel routes.

#### Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-gps-bu353</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

Avaliable for rhiot.version >= 0.1.2


#### URI format

BU353 component supports only consumer endpoints. The BU353 consumer is the polling one, i.e. it periodically asks the GPS device for the
current coordinates. The Camel endpoint URI format for the BU353 consumer is as follows:

    gps-bu353:label

Where `label` can be replaced with any text label:

    from("gps-bu353:current-position").
      convertBodyTo(String.class).
      to("file:///var/gps-coordinates");

BU353 consumer receives the `io.rhiot.component.gps.bu353.ClientGpsCoordinates` instances:

    ClientGpsCoordinates currentPosition = consumerTemplate.receiveBody("gps-bu353:current-position", ClientGpsCoordinates.class);

`ClientGpsCoordinates` class name is prefixed with the `Client` to indicate that these coordinates have been created on the device,
not on the server side of the IoT solution.

#### Options

| Option                   | Default value                                                                 | Description   |
|:-------------------------|:-----------------------------------------------------------------------       |:------------- |
| `consumer.initialDelay`  | 1000                                                                          | Milliseconds before the polling starts. |
| `consumer.delay`         | 5000 | Delay between each GPS scan. |
| `consumer.useFixedDelay` | false | Set to true to use a fixed delay between polls, otherwise fixed rate is used. See ScheduledExecutorService in JDK for details. |
| `coordinatesSource`   | `new SerialGpsCoordinatesSource()`                                               | reference to the`io.rhiot.component.gps.bu353.GpsCoordinatesSource` instance used to read the current GPS coordinates. |

#### Process manager

Process manager is used by the BU353 component to execute Linux commands responsible for starting GPSD daemon or
configuring the GPS receive to provide GPS coordinates in the NMEA mode. If for some reason you would like to change
the default implementation of the process manager used by Camel (i.e. `io.rhiot.utils.process.DefaultProcessManager`),
you can set it on the component level:

    GpsBu353Component bu353 = new GpsBu353Component();
    bu353.setProcessManager(new CustomProcessManager());
    camelContext.addComponent("gps-bu353", bu353);

If custom process manager is not set on the component, Camel will try to find the manager instance in the
[registry](http://camel.apache.org/registry.html). So for example for Spring application, you can just configure
the manager as the bean:

    @Bean
    ProcessManager myProcessManager() {
        new CustomProcessManager();
    }

Custom process manager may be useful if your Linux distribution requires executing some unusual commands
in order to make the GPSD up and running.

#### BU353 type converters

BU353 component comes with the two type converters:
- `String` => `io.rhiot.component.gps.bu353.ClientGpsCoordinates`
- `io.rhiot.component.gps.bu353.ClientGpsCoordinates` => `String`

### Camel GPSD component

Camel [GPSD](http://www.catb.org/gpsd) component can be used to read current GPS information from GPS devices. With Camel GPSD you can
just connect a GPS receiver to your computer's USB port and read the GPS data - the component
will make sure that GPS daemon is up, running and
switched to the [NMEA mode](http://www.gpsinformation.org/dale/nmea.htm). The component also takes care of parsing the
GPSD data read from the server, so you can enjoy the `io.rhiot.component.gps.gpsd.ClientGpsCoordinates`
instances received by your Camel routes.

The GPSD component has been particularly tested against the BU353 GPS unit.
[BU353](http://usglobalsat.com/p-688-bu-353-s4.aspx#images/product/large/688_2.jpg) is one of the most popular and cheapest GPS units on the market.
It is connected to the device via the USB port. If you are looking for good and cheap GPS receiver for your IoT solution, definitely consider purchasing this unit.

#### Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-gpsd</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

#### URI format

The default GPSD consumer is event driven, subscribing to *Time-Position-Velocity* reports and converting them to ClientGpsCoordinates for immediate consumption.
This option offers up to date information, but requires more resources than the scheduled consumer or the producer.

The Camel endpoint URI format for the GPSD consumer is as follows:

    gpsd:label

Where `label` can be replaced with any text label:

    from("gpsd:current-position").
      convertBodyTo(String.class).
      to("file:///var/gps-coordinates");


A scheduled consumer is also supported. This kind of consumer polls GPS receiver every 5 seconds by default. In order to
use it enable the `scheduled` param on the endpoint, just as demonstarted on the snippet below:

    from("gpsd:current-position?scheduled=true").
      convertBodyTo(String.class).
      to("file:///var/gps-coordinates");

Also the GPSD producer can be used to poll a GPS unit "on demand" and return the current location, for example;

    from("jms:current-position").
      to("gpsd:gps");

To subscribe to events or poll a device on another host you have to do two things:
* start GPSD on that host with the param -G to listen on all addresses (eg `gpsd -G /dev/ttyUSB0`)
* pass the host and optionally port to the GPSD endpoint, just as demonstrated on the snippet below:

    from("gpsd:current-position?host=localhost?port=2947").
      convertBodyTo(String.class).
      to("file:///var/gps-coordinates");

The message body is a `io.rhiot.component.gpsd.ClientGpsCoordinates` instance:

    ClientGpsCoordinates currentPosition = consumerTemplate.receiveBody("gpsd:current-position", ClientGpsCoordinates.class);     

`ClientGpsCoordinates` class name is prefixed with the `Client` to indicate that these coordinates have been created on the device,
not on the server side of the IoT solution.

The TPVObject (Time-Position-Velocity report) instance created by a gpsd4java engine is available in exchange under the header
`io.rhiot.gpsd.tpvObject` (constant `io.rhiot.component.gpsd.GpsdConstants.TPV_HEADER`). The original `TPVObject` can be
useful to enrich the payload or do content based routing, eg

    TPVObject tpvObject = exchange.getIn().getHeader(GpsdConstants.TPV_HEADER, TPVObject.class);
    if (tpvObject.getSpeed() > 343) {
        log.info("Camel broke the sound barrier");
    }

#### Options

| Option                   | Default value                                                                 | Description   |
|:-------------------------|:-----------------------------------------------------------------------       |:------------- |
| `host`          | localhost                                                                     | Milliseconds before the polling starts.                                |
| `port`          | 2947                                                                          | Milliseconds before the polling starts.                                |
| `distance`      | 0                                                                             | Distance threshold in km before consuming a message, eg 0.1 for 100m.  |
| `scheduled`     | false                                                                         | Whether the consumer is scheduled or not.  |
| `consumer.initialDelay`  | 1000                                                                          | Milliseconds before the polling starts. Applies only to scheduled consumers. |
| `consumer.delay`         | 5000                                                                          | Delay in milliseconds. Applies only to scheduled consumers.  |
| `consumer.useFixedDelay` | false | Set to true to use a fixed delay between polls, otherwise fixed rate is used. See ScheduledExecutorService in JDK for details. |
| `restartGpsd`         | true                                                                          | Indicates if the endpoint should try (re)start the local GPSD daemon on start.  |
| `gpsd4javaEndpoint`         | `new GPSdEndpoint(host, port, new ResultParser())`                        | Registry reference to the `de.taimos.gpsd4java.backend.GPSdEndpoint` instance to be used by the endpoint.  |


#### Process manager

Process manager is used by the GPSD component to execute Linux commands responsible for starting GPSD daemon or
configuring the GPS receive to provide GPS coordinates in the NMEA mode. If for some reason you would like to change
the default implementation of the process manager used by Camel (i.e. `io.rhiot.utils.process.DefaultProcessManager`),
you can set it on the component level:

    GpsdComponent gpsd = new GpsdComponent();
    gpsd.setProcessManager(new CustomProcessManager());
    camelContext.addComponent("gpsd", gpsd);

If custom process manager is not set on the component, Camel will try to find the manager instance in the
[registry](http://camel.apache.org/registry.html). So for example for Spring application, you can just configure
the manager as the bean:

    @Bean
    ProcessManager myProcessManager() {
        new CustomProcessManager();
    }

Custom process manager may be useful if your Linux distribution requires executing some unusual commands
in order to make the GPSD up and running.


#### Installer

The GPSD component installs it's own dependencies for Debian-based systems using apt-get, these include psmisc, gpsd and gpsd-clients,
as well as their dependencies.
You can configure the installer or set an alternate one on the component:

    GpsdComponent gpsd = new GpsdComponent();
    gpsd.setInstaller(new CustomInstaller());
    camelContext.addComponent("gpsd", gpsd);    

You can also specify alternate/additional dependencies for your platform, if your platform uses my-custom-tools for example, you should configure the component as follows:

    GpsdComponent gpsd = new GpsdComponent();
    gpsd.setRequiredPackages("my-custom-tools,gpsd,gpsd-clients"); //comma-separated list of packages to install
    camelContext.addComponent("gpsd", gpsd);    

If an Installer is not set on the component, Camel will try to find an instance in the
[registry](http://camel.apache.org/registry.html). So for example for Spring application, you can configure the installer as a bean:

    @Bean
    Installer myInstaller() {
        new CustomInstaller();
    }

### Camel Kura router

**Avaliable since Rhiot 0.1.3**: Rhiot provides `io.rhiot.component.kura.router.RhiotKuraRouter` class, which
extends `org.apache.camel.component.kura.KuraRouter` class from the Apache Camel 
[camel-kura](http://camel.apache.org/kura) module. While `KuraRouter` provides a generic base for Kura routes, it doesn't
rely on the Kura-specific jars, because of limitations of the Apache Camel policy regarding adding 3rd parties repositories
to the Camel (like Eclipse Kura repository). `RhiotKuraRouter` extends `KuraRouter` and enhances it with Kura-specific API.

#### Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-kura</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

Adding Rhiot camel-kura module to your project, imports transitive Kura dependencies. This is big advantage over Apache
Camel camel-kura module, which doesn't rely on Kura API and therefore doesn't import Kura jars.


#### Usage

The principle of using `RhiotKuraRouter` is the same as using `KuraRouter` i.e. just extend `RhiotKuraRouter` class:

    import io.rhiot.component.kura.router.RhiotKuraRouter;
    
    class TestKuraRouter extends RhiotKuraRouter {

      @Override
	  public void configure() throws Exception {
	    from("direct:test")
	     .to("mock:test");
	  }

	}

 
### Camel Kura Wifi component

The common scenario for the mobile IoT Gateways, for example those mounted on the trucks or other vehicles, is to cache
collected data locally on the device storage and synchronizing the data with the data center only when trusted WiFi
access point is available near the gateway. Such trusted WiFi network could be localized near the truck fleet parking.
Using this approach, less urgent data (like GPS coordinates stored for the further offline analysis) can be delivered to
the data center without the additional cost related to the GPS transmission fees.

<a href="https://github.com/camel-labs/camel-labs"><img src="images/wifi_truck_1.png" align="center" height="400" hspace="30"></a>
<br>
<a href="https://github.com/camel-labs/camel-labs"><img src="images/wifi_truck_2.png" align="center" height="400" hspace="30"></a>

Camel Kura WiFi component can be used to retrieve the information about the WiFi access spots available within the device
range. Under the hood Kura Wifi component uses Kura `org.eclipse.kura.net.NetworkService`. Kura WiFi component
supports both the consumer and producer endpoints.

#### Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-kura</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

 Avaliable for rhiot.version >= 0.1.1

#### URI format

    kura-wifi:networkInterface/ssid

Where both `networkInterface` and `ssid` can be replaced with the `*` wildcard matching respectively all the network
interfaces and SSIDs.

For example to read all the SSID available near the device, the following route can be used:

    from("kura-wifi:*/*").to("mock:SSIDs");

The Kura WiFi consumer returns the list of the `org.eclipse.kura.net.wifi.WifiAccessPoint` classes returned as a result
of the WiFi scan:

    WifiAccessPoint[] accessPoints = consumerTemplate.receiveBody("kura:wlan0/*", WifiAccessPoint[].class);

You can also request the WiFi scanning using the producer endpoint:

    from("direct:WifiScan").to("kura-wifi:*/*").to("mock:accessPoints");

Or using the producer template directly:

    WifiAccessPoint[] accessPoints = template.requestBody("kura-wifi:*/*", null, WifiAccessPoint[].class);


#### Options

| Option                    | Default value                                                                 | Description   |
|:------------------------- |:-----------------------------------------------------------------------       |:------------- |
| `accessPointsProvider`    | `com.github.camellabs.iot.component.` `kura.wifi.KuraAccessPointsProvider`    | `com.github.camellabs.iot.component.kura.` `wifi.AccessPointsProvider` strategy instance registry reference used to resolve the list of the access points available to consume. |
| `consumer.initialDelay`   | 1000                                                                          | Milliseconds before the polling starts. |
| `consumer.delay`          | 500 | Delay between each access points scan. |
| `consumer.useFixedDelay`  | false | Set to true to use a fixed delay between polls, otherwise fixed rate is used. See ScheduledExecutorService in JDK for details. |

#### Detecting Kura NetworkService

In the first place `io.rhiot.component.kura.wifi.KuraAccessPointsProvider` tries to locate `org.eclipse.kura.net.NetworkService`
in the Camel registry. If exactly one instance of the `NetworkService`  is found (this is usually the case when
if you deploy the route into the Kura container), that instance will be used by the Kura component. Otherwise new instance of the
`org.eclipse.kura.linux.net.NetworkServiceImpl` will be created and cached by the `KuraAccessPointsProvider`.

### Camel Kura PojoSR test facility

This facility is intended to be used to test Camel routes to be deployed on a Kura server. Just instantiate a new `io.rhiot.component.kura.PojosrKuraServer` and call its `start` method passing your Camel route.

	PojosrKuraServer pojosrKuraServer = new PojosrKuraServer();
	MyKuraRouter startedKuraRouter = kura.start(MyKuraRouter.class);

This will instantiate an OSGi registry, add your Camel route and its dependencies to the classpath and execute your route.

### Camel OpenIMAJ component

Camel OpenIMAJ component can be used to detect faces in images.
Camel OpenIMAJ component supports only producer endpoints.

#### Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-openimag</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

 Avaliable for rhiot.version >= 0.1.3

#### URI format

    openimaj:label

Where `label` can be replaced with any text label:

    from("webcam:spycam").to("openimaj:face-detection");

This routes the input stream from the webcam to the openimaj component,
when a face is detected the resulting body will be an instance of org.openimaj.image.processing.face.detection.DetectedFace,
and List<DetectedFace> when there are multiple faces.

The component uses a face detector based on the Haar cascade by default, optionally set an alternate detector;

    from("webcam:spycam").to("openimaj:face-detection?faceDetector=#anotherDetector");

Using the ProducerTemplate, the following example uses the FKEFaceDetector which is a wrapper around the Haar detector, providing additional
information by finding facial keypoints on top of the face;

    KEDetectedFace face = template.requestBody("openimaj:face-detection?faceDetector=#fkeFaceDetector", inputStream, KEDetectedFace.class);
    FacialKeypoint keypoint = face.getKeypoint(FacialKeypoint.FacialKeypointType.EYE_LEFT_LEFT);


The confidence in the face detection is set to the low default of 1, you can set the minimum confidence threshold on the endpoint;

    from("webcam:spycam").to("openimaj:face-detection?confidence=50");


#### Options

| Option                    | Default value                                                                 | Description   |
|:------------------------- |:-----------------------------------------------------------------------       |:------------- |
| `faceDetector`            | `new HaarCascadeDetector()`                                                   | `org.openimaj.image.processing.face.detection.HaarCascadeDetector' is the default face detector |
| `confidence`              | 1                                                                             | The minimum confidence of the detection; higher numbers mean higher confidence.      |



### Camel TinkerForge component

The Camel Tinkerforge component can be used to connect to the TinkerForge brick deamon.

#### Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-tinkerforge</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

 Avaliable for rhiot.version >= 0.1.1

#### General URI format

    tinkerforge:/<brickletType>/<uid>[?parameter=value][&parameter=value]

By default a connection is created to the brickd process running on localhost using no authentication.
If you want to connect to another host use the following format:

    tinkerforge://[username:password@]<host>[:port]/<brickletType>/<uid>[?parameter=value][&parameter=value]

The following values are currently supported as brickletType:

* ambientlight
* temperature
* lcd20x4
* humidity
* io4
* io16
* distance
* ledstrip
* motion
* soundintensity
* piezospeaker
* linearpoti
* rotarypoti
* dualrelay
* solidstaterelay

##### Ambientlight

    from("tinkerforge:/ambientlight/al1")
    .to("log:default");

##### Temperature

    from("tinkerforge:/temperature/T1")
    .to("log:default");

##### Lcd20x4

The LCD 20x4 bricklet has a character based screen that can display 20 characters on 4 rows.

###### Optional URI Parameters

| Parameter | Default value | Description                              |
|-----------|---------------|------------------------------------------|
| line      | 0             | Show message on line 0                   |
| position  | 0             | Show message starting at position 0      |

    from("tinkerforge:/temperature/T1
    .to("tinkerforge:/lcd20x4/lcd1?line=2&position=10

The parameters can be overridden for individual messages by settings them as headers on the exchange:

    from("tinkerforge:/temperature/T1
    .setHeader("line", constant("2"))
    .setHeader("position", constant("10"))
    .to("tinkerforge:/lcd20x4/lcd1");

#### Humidity

     from("tinkerforge:/humidity/H1")
     .to("log:default");

#### Io16

The IO16 bricklet has 2 ports (A and B) which both have 8 IO pins. Consuming and producing
messages happens on port level. So only the port can be specified in the URI and the pin will
be a header on the exchange.

##### Consuming:

    from("tinkerforge:/io16/io9?ioport=a")
    .to("log:default?showHeaders=true");

##### Producing

    from("timer:default?period=2000")
    .setHeader("iopin", constant(0))
    .setHeader("duration", constant(1000))
    .setBody(constant("on"))
    .to("tinkerforge:/io16/io9?ioport=b");


### Camel Pi4j component

Camel Pi4j component can be used to manage GPIO and I2C bus features from Raspberry Pi.
This component uses [pi4j](http://pi4j.com) library

#### Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-pi4j</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

 Avaliable for rhiot.version >= 0.1.1


#### URI format for GPIO

    pi4j-gpio://gpioId[?options]

*gpioId* must match [A-Z_0-9]+ pattern.
By default, pi4j-gpio uses *RaspiPin* Class, change it via *gpioClass* property
You can use static field name "*GPIO_XX*", pin name "*GPIO [0-9]*" or pin address "*[0-9]*"


###### Optional URI Parameters

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

##### Consuming:

    from("pi4j-gpio://13?mode=DIGITAL_INPUT&state=LOW")
    .to("log:default?showHeaders=true");

##### Producing

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


#### URI format for I2C

    pi4j-i2c://busId/deviceId[?options]

###### Optional URI Parameters

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

###### i2c driver

| Driver            | Feature                                                            |
|-------------------|--------------------------------------------------------------------|
| bmp180            | Temperature and Pressure sensor   (http://www.adafruit.com/products/1603) |
| tsl2561           | Light sensor            (http://www.adafruit.com/products/439)     |
| lsm303-accel      | Accelerometer sensor    (http://www.adafruit.com/products/1120)    |
| lsm303-magne      | Magnetometer sensor     (http://www.adafruit.com/products/1120)    |
| mcp23017-lcd      | LCD 2x16 char           (http://www.adafruit.com/products/1109)    |
| hts221      |  Humidity and Temperature sensor used by the [Official RaspberryPi Sense-Hat](https://www.raspberrypi.org/products/sense-hat/) [ST device doc](http://www.st.com/web/en/resource/technical/document/datasheet/DM00116291.pdf)    |
| lps25h      |  Pressure and Temperature sensor used by the [Official RaspberryPi Sense-Hat](https://www.raspberrypi.org/products/sense-hat/) [ST device doc](http://www.st.com/web/en/resource/technical/document/datasheet/DM00066332.pdf)    |


### Camel Framebuffer component

Camel Framebuffer component can be used to manage any Linux Framebuffer.
This component should work with any /dev/fb* linux device.

#### Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-framebuffer</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

 Avaliable for rhiot.version >= 0.1.3


#### URI format for Framebuffer

    framebuffer://name

*name* can /dev/*fbN* or /sys/class/graphics/name content file

###### Optional URI Parameters

| Parameter            | Default value             | Description                                               |
|----------------------|---------------------------|-----------------------------------------------------------|
| `name`               |                           |                                                           |

##### Consuming:

Route that consumes messages from mychannel:

    from("direct:start").
    .to("framebuffer://RPi-Sense FB");

if the body message is empty framebuffer component copies the framebuffer file to the In Body Exchange as byte[].
if the body message not empty framebuffer component copies the In Body Exchange (byte[]) directly to the framebuffer file.

### Camel PubNub component

Camel PubNub component can be used to communicate with the [PubNub](http://www.pubnub.com) data stream network for connected devices.
This component uses [pubnub](https://www.pubnub.com/docs/java/javase/javase-sdk.html) library

#### Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-pubnub</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

 Avaliable for rhiot.version >= 0.1.1


#### General URI format

    pubnub://<pubnubEndpointType>:channel[?options]

The following values are currently supported as pubnubEndpointType:

* pubsub
* presence

###### URI Parameters

| Option                    | Default value                                                                 | Description   |
|:------------------------- |:-----------------------------------------------------------------------       |:------------- |
| `publisherKey`            |                      | The punub publisher key optained from pubnub. Mandatory for publishing events              |
| `subscriberKey`           |                      | The punub subsciber key optained from pubnub. Mandatory when subscribing to events         |
| `secretKey`               |                      | The pubnub secret key.
| `ssl`                     | true                 | Use SSL transport. |
| `uuid`                    |                      | The uuid identifying the connection. If not set it will be auto assigned |
| `operation`               | PUBLISH              | Producer only. The operation to perform when publishing events or ad hoc querying pubnub. Valid values are HERE_NOW, WHERE_NOW, GET_STATE, SET_STATE, GET_HISTORY, PUBLISH |

Operations can be used on the producer endpoint, or as a header:

| Operation                 | Description   |
|:------------------------- |:------------- |
| `PUBLISH`                 | Publish a message to pubnub. The message body shold contain a instance of  `org.json.JSONObject` or `org.json.JSONArray`. Otherwise the message is expected to be a string.
| `HERE_NOW`                | Read presence (Who's online) information from the endpoint channel.|  
| `WHERE_NOW`               | Read presence information for the uuid on the endpoint. You can override that by setting the header `CamelPubNubUUID` to another uuid. |
| `SET_STATE`               | Set the state by uuid. The message body should contain a instance of `org.json.JSONObject` with any state information. By default the endpoint uuid is updated, but you can override that by setting the header `CamelPubNubUUID` to another uuid. |
| `GET_STATE`               | Get the state object `org.json.JSONObject` by for the endpoint uuid. You can override that by setting the `CamelPubNubUUID` header to another uuid. |
| `GET_HISTORY`             | Gets the message history for the endpoint channel. |


##### Consuming:

Route that consumes messages from mychannel:

    from("pubnub://pubsub:mychannel?uuid=master&subscriberKey=mysubkey").routeId("my-route")
    .to("log:default?showHeaders=true");

Route that listens for presence (eg. join, leave, state change) events on a channel

    from("pubnub://presence:mychannel?subscriberKey=mysubkey").routeId("presence-route")
    .to("log:default?showHeaders=true");

##### Producing

Route the collect data and sendt it to pubnub channel mychannel:

    from("timer:default?period=2000").routeId("device-event-route")
    .bean(EventGeneratorBean.class, "getEvent()")
    .convertBodyTo(JSONObject.class)
    .to("pubnub://pubsub:mychannel?uuid=deviceuuid&publisherKey=mypubkey");


### Camel Webcam component

Camel [Webcam](http://webcam-capture.sarxos.pl/) component can be used to capture still images and detect motion.
With Camel Webcam you can connect a camera to your device's USB port, or install the camera mod on the Raspberry Pi board for example,
and poll for an image periodically and respond to motion events.
The body of the message is the current image as a BufferedInputStream, while motion events are stored in the header 'io.rhiot.webcam.webcamMotionEvent'.
This event may be useful for getting the image captured prior to the motion event, as well the Points where the motion occurred and the center of motion gravity.


#### Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-webcam</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

Avaliable for rhiot.version >= 0.1.2

#### URI format


The Camel endpoint URI format for the Webcam consumer is as follows:

    webcam:label

Where `label` can be replaced with any text label:

    from("webcam:spycam").
      to("file:///var/spycam");

This route creates a scheduled consumer taking 1 frame per second and writes it to file in the PNG format.

Alternatively, respond to motion events by setting the endpoint parameter 'motion' to true.

    from("webcam:spycam?motion=true").
      to("file:///var/spycam");

You can also poll the webcam for a single image, for example;

    from("jms:webcam").
      to("webcam:spycam");

Specify the resolution with custom width and height, or the resolution name;

    from("webcam:spycam?resolution=HD720").
      to("seda:cam")

#### Options

| Option                   | Default value                                                                 | Description   |
|:-------------------------|:-----------------------------------------------------------------------       |:------------- |
| `consumer.deviceName`    |                                                                               | Specify the webcam device name, if absent the first device available is used, eg /dev/video0.  |
| `consumer.format`        | PNG                                                                           | Capture format, one of 'PNG,GIF,JPG'  |
| `consumer.initialDelay`  | 1000                                                                          | Milliseconds before the polling starts. Applies only to scheduled consumers. |
| `consumer.motion`        | false                                                                         | Whether to listen for motion events.                                |
| `consumer.motionInterval`| 500                                                                           | Interval in milliseconds to detect motion.                               |
| `consumer.pixelThreshold`| 25                                                                            | Intensity threshold when motion is detected (0 - 255)  |
| `consumer.areaThreshold` | 0.2                                                                           | Percentage threshold of image covered by motion (0 - 100)  |
| `consumer.motionInertia` | -1 (2 * motionInterval)                                                       | Motion inertia (time when motion is valid)  |
| `consumer.resolution`    | QVGA                                                                          | Resolution to use (PAL, HD720, VGA etc)     |
| `consumer.width`         | 320                                                                           | Resolution width, note that resolution takes precendence.  |
| `consumer.height`        | 240                                                                           | Resolution height, note that resolution takes precendence.  |
| `consumer.delay`         | 1000                                                                          | Delay in milliseconds. Applies only to scheduled consumers.  |
| `consumer.useFixedDelay` | false | Set to true to use a fixed delay between polls, otherwise fixed rate is used. See ScheduledExecutorService in JDK for details. |


#### Process manager

Process manager is also used by the Webcam component to execute Linux commands responsible for starting the webcam driver and  
configuring the image format and resolution. If for some reason you would like to change
the default implementation of the process manager used by Camel (i.e. `io.rhiot.utils.process.DefaultProcessManager`),
you can set it on the component level:

    WebcamComponent webcam = new WebcamComponent();
    webcam.setProcessManager(new CustomProcessManager());
    camelContext.addComponent("webcam", webcam);

If custom process manager is not set on the component, Camel will try to find the manager instance in the
[registry](http://camel.apache.org/registry.html). So for example for Spring application, you can just configure
the manager as the bean:

    @Bean
    ProcessManager myProcessManager() {
        new CustomProcessManager();
    }

Custom process manager may be useful if your Linux distribution or camera requires executing some unusual commands
in order to load the v4l2 (Video for Linux) module.


#### Installer

For some Linux+webcam combinations, the webcam component requires `v4l-utils` and its dependencies to be installed on an
operating system. By default the Webcam component installs `v4l-utils` using apt-get, you can configure the installer or
set an alternate one on the component:

    WebcamComponent webcam = new WebcamComponent();
    webcam.setInstaller(new BrewInstaller());
    camelContext.addComponent("webcam", webcam);    

You can also specify alternate dependencies for your platform, if your platform uses my-custom-v4l-utils, configure the component as follows:

    WebcamComponent webcam = new WebcamComponent();
    webcam.setRequiredPackages("my-custom-v4l-utils");  //comma-separated list of packages to install
    camelContext.addComponent("webcam", webcam);    

If an Installer is not set on the component, Camel will try to find an instance in the
[registry](http://camel.apache.org/registry.html). So for example for Spring application, you can configure the installer as a bean:

    @Bean
    Installer myInstaller() {
        CustomInstaller installer = new CustomInstaller();
        installer.setTimeout(60000 * 10); //Allow up to 10 minutes to install packages
    }

By default an installer ignores problems with the webcam packages installation and only logs the warning using a
logger WARN message. If you would like the component to thrown an exception instead of logging a message, set
`ignoreInstallerProblems` property of the `WebcamComponent` to `true`:

    WebcamComponent webcam = new WebcamComponent();
    webcam.setIgnoreInstallerProblems(true);
    camelContext.addComponent("webcam", webcam);


