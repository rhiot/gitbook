# Camera service

Camber service can be used to work with the video images collected by camera devices (such as webcams or surveillance
cameras). Camera service can store and process collected images (for example for image recognition and classification
purposes).

## Camera service API

### Recognizing car registration plate

If you would like to recognize a car registration plate an image, sent the image binary data to the following channel:

    BODY:       <binary>
    HEADERS:    RHIOT_ARG_0 = eu | us
    CHANNEL:    camera.recognizePlate

Where `RHIOT_ARG_0` is a country code of a registration plate you want to recognize (camera service accepts `eu` and `us`
country codes). The result of the invocation is a list of `PlateMatch` objects:

    class PlateMatch {
        String plateNumber;
        double confidence;
    }

Where `confidence` field indicates the percent of confidence of the given image recognition match.

### Processing image offline

If you would like to send image to the camera service for offline processing (including image recognition), send an
image binaries as a message to the following channel:

    BODY:       <binary>
    HEADERS:    RHIOT_ARG_0 = <deviceId>
                RHIOT_ARG_1 = eu | us
    CHANNEL:    camera.process

Where `RHIOT_ARG_0` is a String identifier of a device sending the image data. `RHIOT_ARG_1` is a country code of a
registration plate you want to recognize (camera service accepts `eu` and `us` country codes).

After subiting the image for processing, camera service will save the image to the store location (default is
`/tmp/rhiot/camera` for programmatic runtime, and `/var/rhiot/pass/camera` for PaaS environment). Also a metadata of
the processed image will be stored in a `CameraImage` document collection, using the following schema:

    class CameraImage {
        String id;
        String deviceId;
        PlateMatch[] plateMatches;
    }

## Running camera service in Spring Boot runtime

In order to use camera service in your Spring Boot application add the following jar to your classpath:

	<dependency>
		<groupId>io.rhiot</groupId>
		<artifactId>rhiot-cloudplatform-service-camera</artifactId>
		<version>${rhiot.version}</version>
	</dependency>

Then start your platform:

    import io.rhiot.cloudplatform.runtime.spring.CloudPlatform
    ...
    new CloudPlatform().start();

As soon as platform is started mailbox service will be detected an initialized.
