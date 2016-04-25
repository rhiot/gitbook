# Camera sensor

**Available starting from Rhiot 0.1.5.**

Camera sensor reads data from a camera device and routes that data to the IoT Connector.

By default camera sensor streams image data only when motion is detected. Default motion interval is 250 milisecond i.e.
up to the 4 images per second will be stream to the IoT Connector. The default image format is JPG.

Under the hood camera sensor uses wrapper around
[Raspistill](https://www.raspberrypi.org/documentation/usage/camera/raspicam/raspistill.md) system process. The backend
service responsible for receiving the data from a camera is [Camera Service](../../cloudplatform/services/camera.md).

## Running camera sensor in Spring Boot runtime

In order to enable camera sensor in Spring Boot runtime, add the following jar to your classpath:

    <dependency>
        <groupId>io.rhiot</groupId>
        <artifactId>rhiot-sensor-camera</artifactId>
        <version>${rhiot.version}</version>
    </dependency>

You also have to set `sensor.camera.enabled` property to `true` and specify the identifier of your device using the
`deviceId` property:

    sensor.camera.enabled = true
    deviceId = myDevice

### Work directory

You can configure the work directory of the sensor (i.e. directory where current camera snapshot and queue of images to be
sent to the cloud are stored) using `sensor.camera.workdir` property. Default working directory is `/tmp/camera`.