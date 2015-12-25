# Camel GPSD component

Camel [GPSD](http://www.catb.org/gpsd) component can be used to read current GPS information from GPS devices. With Camel GPSD you can
just connect a GPS receiver to your computer's USB port and read the GPS data - the component
will make sure that GPS daemon is up, running and
switched to the [NMEA mode](http://www.gpsinformation.org/dale/nmea.htm). The component also takes care of parsing the
GPSD data read from the server, so you can enjoy the `io.rhiot.datastream.schema.GpsCoordinates`
instances received by your Camel routes.

The GPSD component has been particularly tested against the BU353 GPS unit.
[BU353](http://usglobalsat.com/p-688-bu-353-s4.aspx#images/product/large/688_2.jpg) is one of the most popular and cheapest GPS units on the market.
It is connected to the device via the USB port. If you are looking for good and cheap GPS receiver for your IoT solution, definitely consider purchasing this unit.

## Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-gpsd</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

## URI format

The default GPSD consumer is event driven, subscribing to *Time-Position-Velocity* reports and converting them to `GpsCoordinates` for immediate consumption.
This option offers up to date information, but requires more resources than the scheduled consumer or the producer.

The Camel endpoint URI format for the GPSD consumer is as follows:

    gpsd:label

Where `label` can be replaced with any text label:

    from("gpsd:current-position").
      marshal().json(Jackson).
      to("file:///var/gps-coordinates");


A scheduled consumer is also supported. This kind of consumer polls GPS receiver every 5 seconds by default. In order to
use it enable the `scheduled` param on the endpoint, just as demonstarted on the snippet below:

    from("gpsd:current-position?scheduled=true").
      marshal().json(Jackson).
      to("file:///var/gps-coordinates");

Also the GPSD producer can be used to poll a GPS unit "on demand" and return the current location, for example;

    from("jms:current-position").
      to("gpsd:gps");

To subscribe to events or poll a device on another host you have to do two things:
* start GPSD on that host with the param -G to listen on all addresses (eg `gpsd -G /dev/ttyUSB0`)
* pass the host and optionally port to the GPSD endpoint, just as demonstrated on the snippet below:

    from("gpsd:current-position?host=localhost?port=2947").
      marshal().json(Jackson).
      to("file:///var/gps-coordinates");

The message body is a `io.rhiot.datastream.schema.GpsCoordinates` instance:

    GpsCoordinates currentPosition = consumerTemplate.receiveBody("gpsd:current-position", GpsCoordinates.class);
    double lat = currentPosition.getLat();
    double lng = currentPosition.getLng();

`GpsCoordinates` class name is prefixed with the `Client` to indicate that these coordinates have been created on the device,
not on the server side of the IoT solution.

The TPVObject (Time-Position-Velocity report) instance created by a gpsd4java engine is available in exchange under the header
`io.rhiot.gpsd.tpvObject` (constant `io.rhiot.component.gpsd.GpsdConstants.TPV_HEADER`). The original `TPVObject` can be
useful to enrich the payload or do content based routing, eg

    TPVObject tpvObject = exchange.getIn().getHeader(GpsdConstants.TPV_HEADER, TPVObject.class);
    if (tpvObject.getSpeed() > 343) {
        log.info("Camel broke the sound barrier");
    }

## Options

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

## Starting the GPSD process

By default, the GPSD component attempts to start the GPSD process every 5 seconds, it does this indefinitely until successfully started and configured.

You can configure limits on the component, eg a maximum of 3 attempts, once per minute:

    GpsdComponent gpsd = new GpsdComponent();
    gpsd.setGpsdMaxRestartAttempts(3); 
    gpsd.setGpsdRestartInterval(60000);
    camelContext.addComponent("gpsd", gpsd);

## Process manager

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
