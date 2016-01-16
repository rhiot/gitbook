# Camel Bluetooth component

Camel Bluetooth component can retrieve information about the bluetooth devices available within the device range. Under the hood Bluetooth component uses bluecove library (http://www.bluecove.org/). Bluetooth component supports both the consumer and producer endpoints.

## Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-bluetooth</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

**Avaliable for `rhiot.version` >= 0.1.3**

## URI format

    bluetooth:label

For example to find all bluetooth devices available near the device, the following route can be used:

    from("bluetooth:scan").to("mock:devices");

The message body is a `io.rhiot.component.bluetooth.BluetoothDevice` instance:

    BluetoothDevices[] devices = consumerTemplate.receiveBody("bluetooth:scan", BluetoothDevices[].class);

You can also request the bluetooth device scanning using the producer endpoint:

    from("direct:scan").to("bluetooth:scan").to("mock:devices");

Or using the producer template directly:

    BluetoothDevices[] devices = template.requestBody("bluetooth:scan", BluetoothDevices[].class);

## Options

| Option                   | Default value                                                                 | Description   |
|:-------------------------|:-----------------------------------------------------------------------       |:------------- |
| `consumer.initialDelay`  | 1000                                                                          | Milliseconds before the polling starts. |
| `consumer.delay`         | 5000 | Delay between each bluetooth device scan. |
| `consumer.useFixedDelay` | false | Set to true to use a fixed delay between polls, otherwise fixed rate is used. See ScheduledExecutorService in JDK for details. |
| `bluetoothDevicesProvider`   | `new BluecoveBluetoothDeviceProvider()`                                               | reference to the`io.rhiot.component.bluetooth.BluetoothDevicesProvider` instance used to scan bluetooth devices. |
| `serviceDiscovery` | false | Search for services on bluetooth device.   


## Installer

The Bluetooth component installs it's own dependencies for Debian-based systems using apt-get, these include blueman and libbluetooth-dev,
as well as their dependencies.
You can configure the installer or set an alternate one on the component:

    BluetoothComponent bluetooth = new BluetoothComponent();
    bluetooth.setInstaller(new CustomInstaller());
    camelContext.addComponent("bluetooth", bluetooth);

You can also specify alternate/additional dependencies for your platform, if your platform uses my-custom-tools for example, you should configure the component as follows:

    BluetoothComponent bluetooth = new BluetoothComponent();
    bluetooth.setRequiredPackages("my-custom-tools,blueman,libbluetooth-dev"); //comma-separated list of packages to install
    camelContext.addComponent("bluetooth", bluetooth);

If an Installer is not set on the component, Camel will try to find an instance in the
[registry](http://camel.apache.org/registry.html). So for example for Spring application, you can configure the installer as a bean:

    @Bean
    Installer myInstaller() {
        new CustomInstaller();
    }
