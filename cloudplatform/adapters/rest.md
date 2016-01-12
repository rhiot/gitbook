# REST protocol adapter

REST protocol adapter bridges HTTP requests with the IoT Connector (i.e. AMQP destinations). It therefore allows you to
expose your services via REST API.

## Protocol binding rules

REST protocol adapter matches HTTP URI to the AMQP queue destination:

    http://example.com:8080/foo/bar => amqp:queue:foo.bar

Content posted with POST request are converted to the body of the message sent to the AMQP destination. JMS correlation ID
can be used to match AMQP response with HTTP response.

## Starting REST protocol adapter

This section describes how to start REST protocol adapter.

### Starting REST protocol adapter in a PaaS environment

PaaS distribution of Cloud Platform has REST protocol adapter included by default (listening on port 8080).

### Starting REST protocol adapter programatically in Spring runtime

In order to start REST protocol adapter in your Cloud Platform application, just add the following jar into your POM file.

    <dependency>
        <groupId>io.rhiot</groupId>
    	<artifactId>rhiot-cloudplatform-adapter-rest</artifactId>
    	<version>${rhiot.version}</version>
    </dependency>

Spring Boot runtime automatically detects and starts REST protocol adapter as soon `CloudPlatform` instance is started:

    new CloudPlatform().start();

## Spring runtime Configuration

By default REST connector listens on HTTP port 8080. You can change this port by setting `rest.port` property.

If you would like to specify the conten ttype returned by the REST protocol adapter, set the value of the `rest.contentType`
property. By default `application/json` is returned.