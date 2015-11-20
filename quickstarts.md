# Quickstarts

Rhiot comes with the set of quickstarts - the sample projects that can be used as the building blocks of your IoT
solution. Quickstarts are hosted at GitHub ([rhiot/quickstarts](https://github.com/rhiot/quickstarts)) and can be
downloaded using the following shell command:

    git clone git@github.com:rhiot/quickstarts.git

## Kura Camel quickstart

The Kura Camel quickstart can be used to create Camel router OSGi bundle project deployable into the
[Eclipse Kura](https://www.eclipse.org/kura) gateway. Kura is a widely adopted field gateway software for the
IoT solutions. Rhiot supports Kura gateway deployments as a first class citizen and this quickstart is intended to be
used as a blueprint for the Camel deployments for Kura. It uses [Camel Kura component](http://camel.apache.org/kura.html)
under the hood.

### Creating a Kura Camel project

In order to create the Kura Camel project execute the following commands:

    git clone git@github.com:rhiot/quickstarts.git
    cp -r quickstarts/kura-camel kura-camel
    cd kura-camel
    mvn install

### Prerequisites

We presume that you have Eclipse Kura already installed on your target device. And that you know the IP address of that device.
If you happen to deploy to a Raspbian-based device, and you would like to find the IP of that Raspberry Pi device connected
to your local network, you can use the Rhiot device scanner, as demonstrated on the snippet below:

    docker run --net=host -it rhiot/deploy-gateway scan

The command above will return an output similar to the one presented below:

    Scanning local networks for devices...

    ======================================
    Device type		IPv4 address
    --------------------------------------
    RaspberryPi2		/192.168.1.100

Keep in mind that `/opt/eclipse/kura/kura/config.ini` file on your target device should have OSGi boot delegation
enabled for packages `sun.*,com.sun.*`. Your `/opt/eclipse/kura/kura/config.ini` should contain the following line then:

    org.osgi.framework.bootdelegation=sun.*,com.sun.*

A boot delegation of `sun` packages is required to make Camel work smoothly in an Equinox.

### Deployment

In order to deploy Camel application to a Kura server, you have to copy necessary Camel jars and a bundle containing your
 application. Your bundle can be deployed into the target device by executing an `scp` command. For example:

    scp target/rhiot-kura-camel-1.0.0-SNAPSHOT.jar pi@192.168.1.100:/tmp

The command above will copy your bundle to the `/tmp/rhiot-kura-camel-1.0.0-SNAPSHOT.jar` location on a target device.
Use similar `scp` command to deploy Camel jars required to run your project:

    scp ~/.m2/repository/org/apache/camel/camel-core/2.16.0/camel-core-2.16.0.jar pi@192.168.1.100:/tmp
    scp ~/.m2/repository/org/apache/camel/camel-core-osgi/2.16.0/camel-core-osgi-2.16.0.jar pi@192.168.1.100:/tmp
    scp ~/.m2/repository/org/apache/camel/camel-kura/2.16.0/camel-kura-2.16.0.jar pi@192.168.1.100:/tmp

Now log into your target device Kura shell using telnet:

    telnet localhost 5002

And install the bundles you previously scp-ed:

    install file:///tmp/camel-core-2.16.0.jar
    install file:///tmp/camel-core-osgi-2.16.0.jar
    install file:///tmp/camel-kura-2.16.0.jar
    install file:///tmp/rhiot-kura-camel-1.0.0-SNAPSHOT.jar

Finally start your application using the following command:

    start <ID_OF_rhiot-kura-camel-1.0.0-SNAPSHOT_BUNDLE)

Keep in mind that bundles you deployed using the recipe above are not installed permanently and will be reverted
after the server restart. Please read Kura documentation for more details regarding
[permanent deployments](http://eclipse.github.io/kura/doc/deploying-bundles.html#making-deployment-permanent).

### What the quickstart is actually doing?

This quickstart triggers [Camel timer](http://camel.apache.org/timer.html) event every second and sends it to the
system logger using [Camel Log](http://camel.apache.org/log) component. This is fairy simple functionality, but enough
to demonstrate the Camel Kura project is actually working and processing messages.

## AMQP cloudlet quickstart

The AMQP cloudlet quickstart can be used as a base for the fat-jar AMQP microservices. If you wanna create a simple
backend application capable of exposing AMQP-endpoint and handling the AMQP-based communication, the AMQT cloudlet
quickstart is the best way to start your development efforts.

### Creating and running the AMQP cloudlet project

In order to create the AMQP cloudlet project execute the following commands:

    git clone git@github.com:rhiot/quickstarts.git
    cp -r quickstarts/cloudlets/amqp amqp
    cd amqp
    mvn install

To start the AMQP cloudlet execute the following command:

    java -jar target/rhiot-cloudlets-amqp-1.0.0-SNAPSHOT.jar

You can also build and run it as a Docker image (we love Docker and highly recommend this approach):

    TARGET_IMAGE=yourUsername/rhiot-cloudlets-amqp
    mvn install docker:build docker:push -Ddocker.image.target=${TARGET_IMAGE}
    docker run -it ${TARGET_IMAGE}

### AMQP broker

By default AMQP cloudlet quickstart starts embedded [ActiveMQ](http://activemq.apache.org) AMQP broker (on
5672 port). If you would like to connect your cloudlet application to the external ActiveMQ broker (instead of starting
the embedded one), run the cloudlet with the `BROKER_URL` environment variable or system property, for example:

    java -DBROKER_URL=tcp://amqbroker.example.com:61616 -jar target/rhiot-cloudlets-amqp-1.0.0-SNAPSHOT.jar

...or...

    docker run -e BROKER_URL=tcp://amqbroker.example.com:61616 -it yourUsername/rhiot-cloudlets-amqp

### Sample chat application

The AMQP cloudlet quickstart is in fact a simple chat application. Clients can send the messages to the chat channel
by subscribing to the broker and sending the messages to the `chat` AMQP queue.

The clients can subscribe to the chat updates
by listening on the `chat-updates` AMQP topic - whenever the new message has been sent to the chat channel, the clients registered
to the `chat-updates` will receive the updated chat history.

The quickstart also exposes the simple REST API that can be used to read the chat history using the HTTP `GET` request:

    $ curl http://localhost:8180/chat
    Hello, this is the IoT device!
    I just wanted to say hello!
    Hello, IoT device. Nice to meet you!

### Architectural overview

When AMQP cloudlet is started with the embedded ActiveMQ broker, the architecture of the example is the following:

<img src="images/quickstarts_cloudlet_amqp_embedded.png" height="400" hspace="30">

When you connect to the external ActiveMQ broker (using `BROKER_URL` option), the architecture of the example becomes
more like the following diagram:

<img src="images/quickstarts_cloudlet_amqp_external.png" height="800" hspace="30">


## MQTT cloudlet quickstart

The MQTT cloudlet quickstart can be used as a base for the fat-jar MQTT microservices.

### Creating and running the MQTT cloudlet project

In order to create the MQTT cloudlet project execute the following commands:

    git clone git@github.com:rhiot/quickstarts.git
    cp -r quickstarts/cloudlets/mqtt mqtt
    cd mqtt
    mvn install

To start the MQTT cloudlet execute the following command:

    java -jar target/rhiot-cloudlets-mqtt-1.0.0-SNAPSHOT.jar

You can also build and run it as a Docker image (we love Docker and recommend this approach):

    TARGET_IMAGE=yourUsername/rhiot-cloudlets-mqtt
    mvn install docker:build docker:push -Ddocker.image.target=${TARGET_IMAGE}
    docker run -it ${TARGET_IMAGE}

### MQTT broker

By default MQTT cloudlet quickstart starts embedded [ActiveMQ](http://activemq.apache.org) MQTT broker (on
1883 port). If you would like to connect your cloudlet application to the external ActiveMQ broker (instead of starting
the embedded one), run the cloudlet with the `BROKER_URL` environment variable or system property, for example:

    java -DBROKER_URL=tcp://amqbroker.example.com:61616 -jar target/rhiot-cloudlets-mqtt-1.0.0-SNAPSHOT.jar

...or...

    docker run -e BROKER_URL=tcp://amqbroker.example.com:61616 -it yourUsername/rhiot-cloudlets-mqtt

### Sample chat application

The MQTT cloudlet quickstart is in fact a simple chat application. Clients can send the messages to the chat channel
by subscribing to the broker and sending the messages to the `chat` MQTT topic. To send some messages to the chat you
can use the standalone [MQTT.js](https://www.npmjs.com/package/mqtt) client:

    mqtt pub -t 'chat' -h 'localhost' -m 'Hello, this is the IoT device!'
    mqtt pub -t 'chat' -h 'localhost' -m 'I just wanted to say hello!'
    mqtt pub -t 'chat' -h 'localhost' -m 'Hello, IoT device. Nice to meet you!'

The clients can subscribe to the chat updates
by listening on the `chat-updates` MQTT topic - whenever the new message has been sent to the chat, the clients registered
to the `chat-updates` will receive the updated chat history.

The quickstart also exposed the simple REST API that can be used to read the chat history using the HTTP `GET` request:

    $ curl http://localhost:8181/chat
    Hello, this is the IoT device!
    I just wanted to say hello!
    Hello, IoT device. Nice to meet you!

### Architectural overview

When MQTT cloudlet is started with the embedded ActiveMQ broker, the architecture of the example is the following:

<img src="images/quickstarts_cloudlet_mqtt_embedded.png" height="400" hspace="30">

When you connect to the external ActiveMQ broker (using `BROKER_URL` option), the architecture of the example becomes
more like the following diagram:

<img src="images/quickstarts_cloudlet_mqtt_external.png" height="800" hspace="30">
