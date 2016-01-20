# Starting Cloud Platform

Rhiot Cloud Platform comes in two supported flavors - running it as a dockerized PaaS or running it programatically using [Spring Boot](http://projects.spring.io/spring-boot/).

## PaaS

PaaS version of Cloud Platform is based on [Docker](https://www.docker.com/) containers. We love Docker and believe that containers are the
future of the applications deployment. To install the Cloud Platform on the Linux server of your choice, just execute the
following command:

    bash <(curl -L -s https://goo.gl/RQd9tJ)

The PaaS script works with any Linux system supporting Docker (including OSX). The script installs the proper version of
Docker server if the latter is not already installed. Keep in mind that the
minimal Docker version required by Cloud Platform is ` >= 1.8.2` - if the older version of the Docker is installed, our
script will upgrade your Docker server. After Docker server is properly installed, Cloud Platform script downloads and starts:

- IoT Connector ([ActiveMQ](http://activemq.apache.org/) broker)
- default protocol adapters server
- default Rhiot backend services server
- standalone Apache Spark cluster (a single master node and a single worker node)
- [MongoDB](https://www.mongodb.org/) database

PaaS Cloud Platform relies on the MongoDB to store some of the data processed by it. For example MongoDB backend is a default
store used by the device service. By default the MongoDB data is stored in the `mongodb_data`
volume container. If such volume doesn't exist, Cloud Platform script will create it for you.


To learn more about Cloud Platform PaaS read [this](paas.md) section of the documentation.

## Using Spring Boot runtime programatically

In order to programatically start Cloud Platform using Spring Boot, add the following dependnecy to your POM file:

	<dependency>
		<groupId>io.rhiot</groupId>
		<artifactId>rhiot-cloudplatform-runtime-spring</artifactId>
		<version>${rhiot.version}</version>
	</dependency>

Then connect to the Cloud Platform:

    import io.rhiot.cloudplatform.runtime.spring.CloudPlatform
    ...
    new CloudPlatform().start();

`CloudPlatform` instance starts its own Spring Boot application context and loads all the extensions (like protocols adapters, encodings and services) available in the classpath.
