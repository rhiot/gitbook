# Performance Testing Framework

The key part of the process of tailoring the perfect IoT solution is choosing the proper hardware for the gateway device.
In general the more expensive gateway hardware is, the more messages per second you can process. However the more expensive the gateway device is, the less affordable your IoT solution becomes for the end client. That is the main reason why would you like to have a proper tool for measuring the given IoT messages flow scenario in the unified way, on many devices.

Rhiot project comes with the *Performance testing framework* that can be used to define the hardware profile and test scenarios. Performance framework takes care of detecting the devices connected to your local network, deploying the test application into these, executing the actual tests and generating the results as the human-readable chart.
For example the sample output for the MQTT QOS testing could generate the following diagram:

<img src="sample_perf_chart.png" align="center" height="400" hspace="30">

Performance Testing Framework excels when you would like to answer the following question - how the different field hardware setups
perform against the given task. Just connect your devices to the local network, execute the performance testing application
and compare the generated diagrams.

## Hardware profiles

This section covers the *hardware profiles* for the performance tests. Profiles are used to describe the particular
hardware configuration that can be used as a target device for the performance benchmark. Every performance test
definition can be executed on the particular hardware profiles.

### Raspberry PI 2 B+ (aka RPI2)

The `RPI2` hardware profile is just the Raspberry Pi 2 B+ model equipped with the network connector (WiFi adapter or
the ethernet cable). Currently we assume that the device is running [Raspbian](https://www.raspbian.org/) Jessie operating
system.

<img src="rpi2_open.jpg" align="center" height="600" hspace="30">
<img src="rpi2_closed.jpg" align="center" height="600" hspace="30">

### Raspberry PI 2 B+ with BU353 (aka RPI2_BU353)

The `RPI2_BU353` hardware profile is the same as `RPI2` profile, but additionally equipped with the
[BU353 GPS receiver](http://usglobalsat.com/p-688-bu-353-s4.aspx#images/product/large/688_2.jpg)
plugged into the USB port.

<img src="rpi2_bu353_open.jpg" align="center" height="500" hspace="30">
<img src="rpi2_bu353_closed.jpg" align="center" height="500" hspace="30">

## Running the performance tester

The easiest way to run the performance benchmark is to connect the target device (for example Rapsberry Pi) into your
local network (for example via the WiFi or the Ethernet cable) and start the tester as a Docker container, using the
following command:

    docker run -v=/tmp/gateway-performance:/tmp/gateway-performance --net=host rhiot/performance-of RPI2

Keep in mind that `RPI2` can be replaced with the other supported hardware profile (like `RPI2_BU353`). The performance tester detects the tests that can be executed for the given hardware profile, deploy the gateway software to the target
device, executes the tests and collects the results.

When the execution of the benchmark ends, the result diagrams will be located in the `/tmp/gateway-performance` directory (or any other directory you specified when executing the command above). The sample diagram may look as follows:

<a href="https://github.com/camel-labs/camel-labs/iot"><img src="sample_perf_chart.png" align="center" height="400" hspace="30"></a>

Keep in mind that currently we assume that your Raspberry Pi has the default Raspbian SSH account available (username: *pi* / password: *raspberry*).

## Analysis of the selected tests results

Below you can find some performance benchmarks with the comments regarding the resulted numbers.

### Mock sensor to the external MQTT broker

In this test we generate mock sensor events in the gateway using the timer trigger. The message is the random UUID encoded
to the array of bytes. Then we send those messages to the
external MQTT broker. We test and compare various MQTT QOS levels. The MQTT client used to send the messages is
[Eclipse Paho](https://www.eclipse.org/paho/).

### Sample results for the RPI2 hardware kit

<img src="RPI2 Mock sensor to external MQTT broker.png" align="center" height="500" hspace="30">

The very first question that comes to the mind when you look at these benchmarks is why there is so huge difference between the MQTT QOS level 0 and the other QOS levels? The reason is that currently Eclipse Paho client doesn't work well with QOS greater than 0 and the high messages load. The reason for that is that Paho client enforces inflight messages limit to 10. This is pretty restrictive treshold considering that MQTT client should have more time for receiving the acknowledgement from the MQTT server. Such acknowledgement is required for the MQTT QOS levels greater than 0. Waiting for the acknowledge reply from the server increases the number of the inflight messages hold by the Paho client. As a result Paho client throughput for QOS 1 and 2 is limited for the extremely large number of messages.

Regardless of the current Paho limits (that are very likely to be changed in the future), the overall performance of the
MQTT client is really great. As the majority of the gateway solutions can safely uses the QOS 0 for forwarding the data from the field to the data center (as losing the single message from the stream of the sensors data, is definitely acceptable).
Almost 1800 messages per second for QOS 0 and around 500 messages per second for the highest QOS 2 is really good result
considering the class of the Raspberry Pi 2 hardware.

