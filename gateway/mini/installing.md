# Installing mini gateway

In order to install Rhiot mini gateway on one of
[supported gateway platforms](https://github.com/rhiot/rhiot/blob/master/docs/readme.md#supported-gateway-platforms)
(like Raspberry Pi with Raspbian) use Rhiot cmd tool
([`rhiot cmd` section describes how to install Rhiot cmd](https://rhiot.gitbooks.io/rhiotdocumentation/content/tooling/cmd.html)).
After Rhiot cmd tool is installed, connect your device to your local network
(using WiFi or the ethernet cable) and execute the following command on a laptop connected to the same network as your
target device:

    rhiot deploy-gateway

From this point forward Rhiot gateway will be installed on your device as `rhiot-gateway` service and started
whenever the device boots up. Under the hood, gateway deployer performs the simple port scanning in the local network
and attempts to connect to supported devices using the default SSH credentials.

To learn more about a gateway deployment tool, see
[`rhiot gateway-deploy` command section](https://rhiot.gitbooks.io/rhiotdocumentation/content/tooling/cmd.html#rhiot-deploygatewa). To learn about
the other useful Rhiot cmd commands, see [`rhiot cmd` section](https://github.com/rhiot/rhiot/blob/master/docs/readme.md#rhiot-command-line-tool-cmd).

## Configuration of the gateway

The gateway configuration file is `/etc/default/rhiot-gateway`. This file is loaded by the gateway
starting script. It means that all the configuration environment variables can be added here. For example to set the
`foo_bar_baz` configuration property to value `qux`, the following environment variable should be added to the
`/etc/default/rhiot-gateway` file:

    export foo_bar_baz=qux

### Gateway logger configuration

By default gateway keeps the last 100 MB of the logging history. Logs are grouped by the days and split into the
10 MB files. The default logging level is `INFO`. You can change it by setting the `camellabs_iot_gateway_log_root_level`
environment variable:

    export camellabs_iot_gateway_log_root_level=DEBUG

## Device heartbeats

Camel gateway generates heartbeats indicating that the device is alive and optionally connected to the data
center.

The default heartbeat rate is 5 seconds, which means that heartbeat events will be generated every 5 second. You
can change the heartbeat rate by setting the `camellabs_iot_gateway_heartbeat_rate` environment variable to the desired
number of the rate miliseconds. The snippet below demonstrates how to change the heartbeat rate to 10 seconds:

    export camellabs_iot_gateway_heartbeat_rate=10000

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
