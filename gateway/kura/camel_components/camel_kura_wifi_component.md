# Camel Kura Wifi component

**This component can be used both in the Kura server and in a regular gateway.**

The common scenario for the mobile IoT Gateways, for example those mounted on the trucks or other vehicles, is to cache collected data locally on the device storage and synchronizing the data with the data center only when trusted WiFi access point is available near the gateway. Such trusted WiFi network could be localized near the truck fleet parking.
Using this approach, less urgent data (like GPS coordinates stored for the further offline analysis) can be delivered to the data center without the additional cost related to the GPS transmission fees.

<img src="../../images/wifi_truck_1.png" align="center" height="400" hspace="30">
<br>
<img src="../../images/wifi_truck_2.png" align="center" height="400" hspace="30">

Camel Kura WiFi component can be used to retrieve the information about the WiFi access spots available within the device
range. Under the hood Kura Wifi component uses Kura `org.eclipse.kura.net.NetworkService`. Kura WiFi component
supports both the consumer and producer endpoints.

## Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-kura</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

 Avaliable for rhiot.version >= 0.1.1

## URI format

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


## Options

| Option                    | Default value                                                                 | Description   |
|:------------------------- |:-----------------------------------------------------------------------       |:------------- |
| `accessPointsProvider`    | `com.github.camellabs.iot.component.` `kura.wifi.KuraAccessPointsProvider`    | `com.github.camellabs.iot.component.kura.` `wifi.AccessPointsProvider` strategy instance registry reference used to resolve the list of the access points available to consume. |
| `consumer.initialDelay`   | 1000                                                                          | Milliseconds before the polling starts. |
| `consumer.delay`          | 500 | Delay between each access points scan. |
| `consumer.useFixedDelay`  | false | Set to true to use a fixed delay between polls, otherwise fixed rate is used. See ScheduledExecutorService in JDK for details. |

## Detecting Kura NetworkService

In the first place `io.rhiot.component.kura.wifi.KuraAccessPointsProvider` tries to locate `org.eclipse.kura.net.NetworkService`
in the Camel registry. If exactly one instance of the `NetworkService`  is found (this is usually the case when
if you deploy the route into the Kura container), that instance will be used by the Kura component. Otherwise new instance of the
`org.eclipse.kura.linux.net.NetworkServiceImpl` will be created and cached by the `KuraAccessPointsProvider`.



