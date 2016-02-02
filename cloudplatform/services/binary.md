# Binary service

Binary service can be used to store binary payload in a dedicated pluggable binary store. This functionality is primarily
used by services like [Camera service](camera.md) to store and serve images captured by video cameras.

## Service API

To save binary array via the service, send those binaries to the following channel:

    BODY:       <binary>
    HEADERS:    RHIOT_ARG_0 = String id
    CHANNEL:    binary.store

Where `RHIOT_ARG_0` is am unique identifier of the given binary resource. To read a binary from a store, send a request
to the following channel:

    BODY:       String id
    CHANNEL:    binary.read
    RESPONSE:   byte[] binary

## Binary storage

Binary service will save the data to the store location (default is
`/tmp/rhiot/camera` directory for programmatic runtime, and `/var/rhiot/pass/camera` for PaaS environment).

## Running binary service in Spring Boot runtime

In order to use camera service in your Spring Boot application add the following jar to your classpath:

	<dependency>
		<groupId>io.rhiot</groupId>
		<artifactId>rhiot-cloudplatform-service-binary</artifactId>
		<version>${rhiot.version}</version>
	</dependency>

Then start your platform:

    import io.rhiot.cloudplatform.runtime.spring.CloudPlatform
    ...
    new CloudPlatform().start();

As soon as platform is started mailbox service will be detected an initialized.
