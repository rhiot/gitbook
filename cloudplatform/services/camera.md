# Camera service

Camber service can be used to work with the video images collected by camera devices (such as webcams, drone cameras or
surveillance cameras). Camera service can store and process collected images (for example for image recognition and classification
purposes).

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

As soon as platform is started camera service will be detected an initialized.

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

After subiting the image for processing, camera service will save the image using [Binary Service](binary.md). Also a
metadata of a processed image will be stored in a `CameraImage` document collection, using the following schema:

    class CameraImage {
        String id;
        String deviceId;
        PlateMatch[] plateMatches;
    }

## Rotation of an image data

Every minute camera service executes internal rotation task which deletes the oldest camera image binary data together
with its metadata (an appropriate `CameraImage` document). Each run of the rotation task removes 1000 items.

The task will be executed only when estimated total size of the images is greater than a storage quota. Images are
assumed to have `10 KB` size. The default quota size is `5 GB`.

### Configuration of an image rotation task

To change default `5 GB` storage quota set `camera.rotation.storageQuota` property. The quota is defined in megabytes.

    camera.rotation.storageQuota: 1024 # 1 GB

## Offline image processing

Image data submitted to the `camera.process` operation are processed asynchronously by multiple processors in order to
extract additional meta data from an image (for example to tag an image with detected license plate numbers).

### OpenALPR license plate recognition

OpenALRP processor analyzes submitted images against a presence of license plates. Detected license plates are stored
in a `CameraImage` document collection.

If you would like to disable OpenALPR image processor set `camera.processor.openalpr.enabled` to `false`:

    camera.processor.openalpr.enabled = false
