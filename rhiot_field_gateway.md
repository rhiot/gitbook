# Rhiot field gateway

## Rhiot field gateway

*Rhiot gateway* is the small fat jar application that can be installed into the field device. Gateway acts as a bridge between the sensors and the data center. Under the hood, Rhiot gateway is the fat jar running
[Vert.x](http://vertx.io) and [Apache Camel](http://camel.apache.org).

### Installing gateway

In order to install Rhiot gateway on one of
[supported gateway platforms](https://github.com/rhiot/rhiot/blob/master/docs/readme.md#supported-gateway-platforms)
(like Raspberry Pi with Raspbian) use Rhiot cmd tool
([`rhiot cmd` section describes how to install Rhiot cmd](https://github.com/rhiot/rhiot/blob/master/docs/readme.md#rhiot-command-line-tool-cmd)).
After Rhiot cmd tool is installed, connect your device to your local network
(using WiFi or the ethernet cable) and execute the following command on a laptop connected to the same network as your
target device:

    rhiot deploy-gateway

From this point forward Rhiot gateway will be installed on your device as `rhiot-gateway` service and started
whenever the device boots up. Under the hood, gateway deployer performs the simple port scanning in the local network
and attempts to connect to supported devices using the default SSH credentials.

To learn more about a gateway deployment tool, see
[`rhiot gateway-deploy` command section](https://github.com/rhiot/rhiot/blob/master/docs/readme.md#rhiot-deploy-gateway). To learn about
the other useful Rhiot cmd commands, see [`rhiot cmd` section](https://github.com/rhiot/rhiot/blob/master/docs/readme.md#rhiot-command-line-tool-cmd).

### Configuration of the gateway

The gateway configuration file is `/etc/default/rhiot-gateway`. This file is loaded by the gateway starting script. It means that all the configuration environment variables can be added here. For example to set the
`foo_bar_baz` configuration property to value `qux`, the following environment variable should be added to the
`/etc/default/rhiot-gateway` file:

    export foo_bar_baz=qux

### Gateway logger configuration

By default gateway keeps the last 100 MB of the logging history. Logs are grouped by the days and split into the
10 MB files. The default logging level is `INFO`. You can change it by setting the `camellabs_iot_gateway_log_root_level`
environment variable:

    export camellabs_iot_gateway_log_root_level=DEBUG

### Device heartbeats

Camel gateway generates heartbeats indicating that the device is alive and optionally connected to the data
center.

The default heartbeat rate is 5 seconds, which means that heartbeat events will be generated every 5 second. You
can change the heartbeat rate by setting the **camellabs_iot_gateway_heartbeat_rate** environment variable to the desired
number of the rate miliseconds. The snippet below demonstrates how to change the heartbeat rate to 10 seconds:

```
  export camellabs_iot_gateway_heartbeat_rate=10000
```

The heartbeat events are broadcasted to the Vert.x event bus address `heartbeat` (
`io.rhiot.gateway.GatewayConstants.BUS_HEARTBEAT` constant).

#### Logging heartbeat

By default Camel gateway sends the heartbeat event to the application log (at the `INFO` level). Logging heartbeats are
useful when verifying that gateway is still running - you can just take a look into the application log files. The name of the logger is `io.rhiot.gateway.heartbeat.LoggingHeartbeatVerticle`
and the message is `Ping!`.

#### MQTT heartbeat

Camel IoT gateway can send the heartbeat events to the data center using the MQTT protocol. MQTT heartbeats are
useful when verifying that gateway is still running and connected to the backend services deployed into the data
center.

In order to enable the
MQTT-based heartbeats set the `camellabs.iot.gateway.heartbeat.mqtt` environment variable to `true`. Just as
demonstrated on the snippet below:

    export camellabs.iot.gateway.heartbeat.mqtt=true

The address of the target MQTT broker can be set using the `camellabs.iot.gateway.heartbeat.mqtt.broker.url` environment
variable, just as demonstrated on the example below:

    export camellabs.iot.gateway.heartbeat.mqtt.broker.url=tcp://mydatacenter.com

By default MQTT heartbeat sends events to the `heartbeat` topic. You can change the name of the topic using the
`camellabs.iot.gateway.heartbeat.mqtt.topic` environment variable. For example to send the heartbeats to the
`myheartbeats` queue set the `camellabs.iot.gateway.heartbeat.mqtt.topic` environment variable as follows:

    export camellabs.iot.gateway.heartbeat.mqtt.topic=myheartbeats

The heartbeat message format is `hostname:timestamp`, where `hostname` is the name of the device host and `timestamp` is
the current time converted to the Java miliseconds.

#### LED heartbeat

Heartbeats can blink the LED node connected to the gateway device. This is great visual indication that the gateway
software is still up and running. For activating LED heartbeat set `camellabs.iot.gateway.heartbeat.led` property to `true`.
Like this

    export camellabs.iot.gateway.heartbeat.led=true

The LED output port can be set via `camellabs.iot.gateway.heartbeat.led.gpioId` environment variable, Default value is 0 *wiring lib pin index*
Change LED output port like this:

    export camellabs.iot.gateway.heartbeat.led.gpioId=11

Please add resistor (220 Ohms) between LED and Mass (0v) to avoid excessive current through it.

### Using GPS receivers with gateway

Rhiot gateway comes with a dedicated GPS support. It makes it easier to collect, store and forward the GPS data to a
data center (to the [Rhiot Cloud](https://github.com/rhiot/rhiot/blob/master/docs/readme.md#rhiot-cloud) in particular).
Under the hood gateway GPS support uses the [GPSD component](https://github.com/rhiot/rhiot/blob/master/docs/readme.md#camel-gpsd-component).

#### Enabling GPS receiver

In order to start collecting GPS data on the gateway device, just add `gps=true` property to the gateway configuration.
Gateway with the GPS support enabled will poll the GPS receiver every 5 seconds and store the GPS coordinates in the
`/var/rhiot/gps` directory (or the other directory indicated using `gps_store_directory` configuration property). The data
collected from the GPS receiver will be serialized to the JSON format. Each GPS read is saved to in a dedicated file.

The JSON schema of the collected coordinates is similar to teh following example:

    { 
      "timestamp": 1445356703704,
      "lat": 20.0, 
      "lng": 30.0
    }

#### Enriching GPS data

Sometimes you would like to collect not only the GPS coordinates, but some other sensor data associated with the given location. For example to track the temperature or WiFi networks available in various locations. If you would like to enrich collected GPS data with value read from some other endpoint, set the `gps_enrich` property to the Camel endpoint
URI value. For example setting `gps_enrich=kura-wifi:*/*` can be used to poll for available WiFi networks using
[Camel Kura WiFi component](https://github.com/rhiot/rhiot/blob/master/docs/readme.md#camel-kura-wifi-component) and adding the results to collected GPS payloads.

The enriched JSON schema of the collected coordinates is similar to teh following example:

    {"timestamp": 1445356703704,
      "lat": 20.0, "lng": 30.0,
      "enriched": {
        "foo": "bar"
      }
    }      

#### Sending collected GPS data with a data center

If you would like to synchronize the collected GPS data with a data center, consider using the out-of-the-box functionality that Rhiot gateway offers. In order to enable an automatic synchronization of the GPS coordinates, set the
`gps_cloudlet_sync` configuration property to `true`.

The gateway synchronization route reads GPS messages saved in the gateway store (see the section above) and sends those to the destination endpoint. The target Camel endpoint can be specified using the `gps_cloudlet_endpoint` configuration property. For example the following property tells a gateway to send the GPS data to a MQTT broker:

    gps_cloudlet_endpoint=paho:topic?brokerUrl=tcp://broker.com:1234

The default data center endpoint is Netty REST call (using POST method) - `netty4-http:http://${cloudletAddress}/api/document/save/GpsCoordinates`, where
`gps_cloudlet_address` configuration property has to be configured to match your HTTP API address (for example
`gps_cloudlet_address=myapi.com:8080`).

The default data format used by the gateway is JSON, where the message schema is similar to the folowwing example:

    { 
      "client": "my-device",
       "clientId": "local-id-generated-on-client",
       "timestamp": 1445356703704,
       "latitude": 20.0,
       "longitude": 30.0,
       "enriched": {
         "foo": "bar"
       }
     }

### Monitoring gateway with Jolokia

The gateway exposes its JMX beans using the [Jolokia REST API](https://jolokia.org). The default Jolokia URL for the
gateway is `http://0.0.0.0:8778/jolokia`. You can take advantage of the Jolokia to monitor and perform administrative
tasks on your gateway.

### Adding the custom code to the gateway

Camel IoT gateway comes with the set of predefined components and features that can be used out of the box. It is
however very likely that your gateway will execute some custom logic related to your business domain. This section of
the documentation covers how can you add the custom code to the gateway.

We highly recommend to deploy the gateway as a fat jar. This approach reduces the devOps cycles needed to deliver the
working solution. If you would like to add the custom logic to the gateway, we recommend to create the fat jar Maven
project including the gateway core dependency:

    <dependencies>
      <dependency>
        <groupId>io.rhiot</groupId>
        <artifactId>rhiot-gateway</artifactId>
        <version>${rhiot.version}</version>
      </dependency>
    </dependencies>

Avaliable for rhiot.version >= 0.1.1

Now all your custom code can just be added to the project. The resulting fat jar will contain both gateway core logic
and your custom code.

#### Adding custom Groovy Camel verticle to the gateway

As the Rhiot gateway uses Vert.x event bus as its internal messaging core, the recommended option to add new Camel routes to the
gateway is to deploy those as the Vert.x verticle. The Vert.x helper classes for the gateway are available in the
following jar:

    <dependencies>
      <dependency>
        <groupId>io.rhiot</groupId>
        <artifactId>rhiot-vertx</artifactId>
        <version>${rhiot.version}</version>
      </dependency>
    </dependencies>

 Avaliable for rhiot.version >= 0.1.1

In order to create Vert.x verticle that can access single `CamelContex` instance shared between all the verticles
within the given JVM, extend the `io.rhiot.vertx.camel.GroovyCamelVerticle` class:

    @GatewayVerticle
    class HeartbeatConsumerVerticle extends GroovyCamelVerticle {

        @Override
        void start() {
            fromEventBus('heartbeat') { it.to('mock:camelHeartbeatConsumer') }
        }

    }

Rhiot gateway scans the classpath for the verticle classes marked with the `io.rhiot.gateway.GatewayVerticle`
annotation. All those verticles are automatically loaded into the Vert.x backbone.

As you can see in the example above you can read the messages from the event bus and forward these to your Camel
route using the `fromEventBus(channel, closure(route))` method. You can also access the Camel context directly:

    import static io.rhiot.steroids.camel.CamelBootInitializer.camelContext

    @GatewayVerticle
    class HeartbeatConsumerVerticle extends GroovyCamelVerticle {

        @Override
        void start() {
            camelContext().addRoutes(new RouteBuilder(){
                @Override
                void configure() {
                    from('timer:test').to('seda:test')
                }
            })
        }

    }
