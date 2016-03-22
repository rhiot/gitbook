# Device sensor

**Available starting from Rhiot 0.1.5.**

Device sensor is responsible for providing device metadata to the cloud. The latter includes the registration 
of device at stratup and heartbeats updates.

The backend
service responsible for receiving the data from device sensor is [device management service](../../cloudplatform/services/device_management_.md).

## Enabling camera sensor in Spring Boot runtime

In order to enable camera sensor in Spring Boot runtime, add the following jar to your classpath:

    <dependency>
        <groupId>io.rhiot</groupId>
        <artifactId>rhiot-sensor-device</artifactId>
        <version>${rhiot.version}</version>
    </dependency>

### Registration of a device

On the startup of the gateway application device sensor registers itself using the 
[device management service](../../cloudplatform/services/device_management_.md). The
identifier of a device should be specified using `deviceId` property:

    deviceId = myDevice
