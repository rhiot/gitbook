# AMQP Cloudlet quickstarts

The AMQP cloudlet quickstart can be used as a base for the fat-jar AMQP microservices. If you wanna create a simple
backend application capable of exposing AMQP-endpoint and handling the AMQP-based communication, the AMQT cloudlet
quickstart is the best way to start your development efforts.

#### Creating and running the AMQP cloudlet project

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

#### AMQP broker

By default AMQP cloudlet quickstart starts embedded [ActiveMQ](http://activemq.apache.org) AMQP broker (on
5672 port). If you would like to connect your cloudlet application to the external ActiveMQ broker (instead of starting
the embedded one), run the cloudlet with the `BROKER_URL` environment variable or system property, for example:

    java -DBROKER_URL=tcp://amqbroker.example.com:61616 -jar target/rhiot-cloudlets-amqp-1.0.0-SNAPSHOT.jar

...or...

    docker run -e BROKER_URL=tcp://amqbroker.example.com:61616 -it yourUsername/rhiot-cloudlets-amqp

#### Sample chat application

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

#### Architectural overview

When AMQP cloudlet is started with the embedded ActiveMQ broker, the architecture of the example is the following:

<img src="quickstarts_cloudlet_amqp_embedded.png" height="400" hspace="30">

When you connect to the external ActiveMQ broker (using `BROKER_URL` option), the architecture of the example becomes
more like the following diagram:

<img src="quickstarts_cloudlet_amqp_external.png" height="800" hspace="30">

