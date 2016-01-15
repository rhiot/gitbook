# Device management service

The foundation of the every IoT solution is the device management system. Without the centralized coordination of your
*things*, you can't properly orchestrate how your devices communicate with each other. Also the effective monitoring of
the IoT system, without the devices registered in the centralized cloud, becomes almost impossible. Rhiot provides
backend management service for registering and tracking devices connected to the Cloud Platform.

The diagram below presents the high-level overview of the device cloudlet architecture.

<img src="rhiot_cloud_platform_devices.png" align="center" height="600">

## Device management API

Device service can be accessed using the IoT connector API described below. Keep in mind that Protocol Adapters can
be used to access device management service via other protocols (including REST and LWM2M).

### Device management schema

Devices are represented using the following schema:

    Device {

    String deviceId;

    String registrationId;

    Date registrationDate;

    Date lastUpdate;

    InetAddress address;

    int port;

    InetSocketAddress registrationEndpointAddress;

    long lifeTimeInSec;

    String smsNumber;

    String lwM2mVersion;

    BindingMode bindingMode;

    }

### Registering device

In order to register a device in a device service, send a definition of a former to the `device.register` channel:

    device -> device.register

Where `device` is an encoded object following the `Device` schema.

### Updating device metadata

In order to updated already registered device, send a definition of it to the `device.update` channel:

    device -> device.update

Where `device` is an encoded object following the `Device` schema. Keep in mind that `deviceId` and `registrationId`
fields of the encoded payload must match the corresponding values of the device registered in a device service.

### Listing devices

To list the devices registered to the cloud (together with their basic metadata) send empty message to the following
IoT Connector channel:

    device.list

As a response you will receive a list of the devices following the `Device` schema described above.

### Reading particular device's metadata

In order to read the metadata of the particular device identified with the given ID, send a message to following
IoT connector channel:

    String id -> device.get

Where ID is a String representing the unique ID of a device. As a response you will receive a device following the
`Device` schema described above.

If you would like to find device using its unique registration identified, assigned to it when device is connected to
the device service, the `device.getByRegistrationId` channel instead:

    String registrationID -> device.getByRegistrationId

### Disconnected devices

Registered devices can be in the *connected* or *disconnected* status. Connected devices can exchange
messages with the cloud, while disconnected can't. Disconnection usually occurs when device temporarily lost the network
connectivity.

To return the list of identifiers of the disconnected devices send an empty message to the following IoT connector channel:

    device.disconnected

In the response you will receive the list of the identifiers of the devices
that have not send the heartbeat signal to the device management service for the given *disconnection period* (one minute by
default).

### Sending heartbeat

The device which is running and operational should periodically send the hearbeat signal to the device service in order to avoid
being marked as disconnected. You can do it be sending a message to the following IoT connector channel:

    String id -> device.heartbeat

Where ID is a String representing the unique ID of a device.

Keep also in mind that sending the regular LWM2M update by the client device to the LWM2M Leshan protocol adapter sends
heartbeat update as well.

### Deregistering device

Sometimes you would like to explicitly remove the particular registered device from a device database. In such case
you can send a message to the following IoT connector channel:

    String id -> device.deregister

Where ID is a String representing the unique ID of a device.

## Running device service in Spring Boot runtime

The disconnection period can be changed globally using the `disconnectionPeriod` environment variable indicating the
disconnection period value in miliseconds. For example the snippet below sets the disconnection period to 20 seconds:

    disconnectionPeriod=20000


---

##### Reading device's details

LWM2M protocol allows you to read the values of the various metrics from the managed device. The basic metrics includes
device's manufacturer name, model, serial number, firmware version and so forth. In order to read the device details,
send `GET` request to the `/device/myDeviceID/details` URI. For example to read the details of the device
identified by the `myDeviceID`, execute the following command:

    $ curl http://rhiot.net:15000/device/myDeviceID/details
    {"deviceDetails":
      {"serialNumber":"Serial-0cc28150-3a09-4acc-b12d-d9101b8a29d2",
      "modelNumber":"Virtual device",
      "firmwareVersion":"1.0.0",
      "manufacturer":"Rhiot"}
    }

The `/device/ID/details` call connects to the given device, collects the metrics and returns those metrics wrapped into
the JSON response. You can also collect individual metrics using the following URIs:

| Metric                   | URI                      | Description   |
|:-------------------------|:-------------------------       |:------------- |
| Manufacturer | `/device/deviceId/manufacturer`                | Device's manufacturer. For example `Raspberry Pi`. |
| Model number         | `/device/deviceId/modelNumber` | Device's model. For example `2 B+`. |
| Serial number | `/device/deviceId/serialNumber` | Unique string identifying the particular piece of the hardware. |
| Firmware number   | `/device/deviceId/firmwareVersion`  | The text identifying the version of the software that device is running. Can include both operating system and applications' versions. |

For example to read the version of the software used by your device, execute the following `GET` request:

     $ curl http://rhiot.net:15000/device/foo/firmwareVersion
    {"firmwareVersion":"1.0.0"}

Keep in mind that the metric values read by these operations are saved to the metrics database and can be accessed later on
(see [Devices Data Analytics](#devices-data-analytics)). Also if
the device is disconnected at the moment when the REST API is called, the value will be read from the metrics history database,
instead of the real device. If there is no historical value available for the given device and metric, the
`unknown - device disconnected` value will be returned for it. For example:

     $ curl http://rhiot.net:15000/device/foo/firmwareVersion
    {"firmwareVersion":"unknown - device disconnected"}

##### Creating virtual devices

Device Cloudlet offers you the option to create the *virtual devices*. Virtual devices can be used to represent the clients which
can't use LWM2M API. For example the Android phone could use REST calls only to create and maintain the projection of
itself as the virtual device even if you can't install LWM2M client on that device.

To create the virtual device, send the `POST` request to the `/client` URI. For example:

    curl -X POST -d '{ "clientId": "myVirtualDeviceId"}' http://rhiot.net:15000/client
    {"Status":"Success"}%

Starting from this point your the virtual device identified as `myVirtualDeviceId` will be registered in the device
cloudlet LWM2M server. And of course you can send the heartbeats signals to the virtual device to indicate that the
device is still connected to the Rhiot Cloud.

    $ curl http://rhiot.net:15000/device/myDeviceID/heartbeat
    {"status": "success"}

#### Devices data analytics

LWM2M protocol provides you the way to read the metrics' values from the devices. However in order to perform the search
queries against those values, you have to store those in a centralized storage. For example if you would like to find all the
devices with the firmware version smaller than `x.y.z`, you have to store all the firmware version of your devices in
the centralized database, then execute a query against that database. Otherwise you will be forced to connect to the all of
your devices using the LWM2M protocol and ask each device to provide its firmware version number. Asking millions of
the devices connected to your system to provide you their firmware version is far from the ideal in the terms of the
efficiency.

The device cloudlet stores each device metric value read from the LWM2M server via the REST API in the dedicated analytics
store. It basically means that whenever you call the REST API to read the device metric, the value read from the
device is stored in the database. For example the following API call will not only return the firmware version of the
device identified by `myDevice`, but also will remember this value in the analytics database:

    $ curl http://rhiot.net/device/myDevice/firmwareVersion
    {"firmwareVersion": "1.0.0"}

By default the historical metrics data is saved in the [MongoDB](https://www.mongodb.org) database. The default database name is
`DeviceCloudlet`, while the historical values of the read metrics are saved to the `DeviceMetrics` collection. Device
Cloudlet reuses the MongoDB connection settings used by the MongoDB LWM2M device registry store.
