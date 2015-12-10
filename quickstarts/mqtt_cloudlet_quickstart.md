# MQTT Cloudlet quickstart

The MQTT cloudlet quickstart can be used as a base for the fat-jar MQTT microservices.

#### Creating and running the MQTT cloudlet project

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

#### MQTT broker

By default MQTT cloudlet quickstart starts embedded [ActiveMQ](http://activemq.apache.org) MQTT broker (on
1883 port). If you would like to connect your cloudlet application to the external ActiveMQ broker (instead of starting
the embedded one), run the cloudlet with the `BROKER_URL` environment variable or system property, for example:

    java -DBROKER_URL=tcp://amqbroker.example.com:61616 -jar target/rhiot-cloudlets-mqtt-1.0.0-SNAPSHOT.jar

...or...

    docker run -e BROKER_URL=tcp://amqbroker.example.com:61616 -it yourUsername/rhiot-cloudlets-mqtt

#### Sample chat application

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

#### Architectural overview

When MQTT cloudlet is started with the embedded ActiveMQ broker, the architecture of the example is the following:

<img src="quickstarts_cloudlet_mqtt_embedded.png" height="400" hspace="30">

When you connect to the external ActiveMQ broker (using `BROKER_URL` option), the architecture of the example becomes
more like the following diagram:

<img src="quickstarts_cloudlet_mqtt_external.png" height="800" hspace="30">
