# Kura Camel quickstart

The Kura Camel quickstart can be used to create Camel router OSGi bundle project deployable into the
[Eclipse Kura](https://www.eclipse.org/kura) gateway. Kura is a widely adopted field gateway software for the
IoT solutions. Rhiot supports Kura gateway deployments as a first class citizen and this quickstart is intended to be used as a blueprint for the Camel deployments for Kura. It uses [Camel Kura component](http://camel.apache.org/kura.html) under the hood.

## Creating a Kura Camel project

In order to create the Kura Camel project execute the following commands:

    git clone git@github.com:rhiot/quickstarts.git
    cp -r quickstarts/kura-camel kura-camel
    cd kura-camel
    mvn install

## Prerequisites

### Find your device

We presume that you have [Eclipse Kura](https://wiki.eclipse.org/Kura/Raspberry_Pi) already installed on your target device. And that you know the IP address of that device.

Then export address of your Raspberry Pi device to RPBI IP environment variable:

    export RBPI_IP=192.168.1.100

### Configure Kura 

Keep in mind that `/opt/eclipse/kura/kura/config.ini` file on your target device should have OSGi boot delegation
enabled for packages `sun`. A boot delegation of `sun` packages is required to make Camel work smoothly in 
[Eclipse Equinox](http://www.eclipse.org/equinox/). In order to enable boot delegation, just add the following line to
the `/opt/eclipse/kura/kura/config.ini`:

    org.osgi.framework.bootdelegation=sun.*,com.sun.*

## Deployment

In order to deploy Camel application to a Kura server, you have to copy necessary Camel jars and a bundle containing your application. Your bundle can be deployed into the target device by executing an `scp` command. For example:

    scp target/rhiot-kura-camel-1.0.0-SNAPSHOT.jar pi@${RBPI_IP}:

The command above will copy your bundle to the `/home/pi/rhiot-kura-camel-1.0.0-SNAPSHOT.jar` location on a target device.

Now log into your target device using SSH. Then, from a remote SSH shell, log into Kura shell using telnet:

    ssh pi@${RBPI_IP}
    ...
    telnet localhost 5002

And install the bundles you previously scp-ed into the telnet session :

    install file:///home/pi/rhiot-kura-camel-1.0.0-SNAPSHOT.jar

Finally start your application using the following command:

    start < ID_OF_rhiot-kura-camel-1.0.0-SNAPSHOT_BUNDLE >

You can retrieve ID of your bundle using the following command:

    ss | grep rhiot-kura-camel

Keep in mind that bundles you deployed using the recipe above are not installed permanently and will be reverted after the server restart. Please read Kura documentation for more details regarding
[permanent deployments](http://eclipse.github.io/kura/doc/deploying-bundles.html#making-deployment-permanent).

## What the quickstart is actually doing?

This quickstart triggers [Camel timer](http://camel.apache.org/timer.html) event every second and sends it to the system 
logger using [Camel Log](http://camel.apache.org/log) component. This is fairy simple functionality, but enough to 
demonstrate the Camel Kura project is actually working and processing messages.

In order to see messages logged by Camel router to Kura log file execute the following command on your remote device:

    tail -f /var/log/kura.log

## What else can I do with it?

Go to the [Camel Kura router documentation page](../gateway/camel_kura_router.md) to learn more about Camel and Kura
awesomeness. For example you can [modify your XML routes from the web UI ](../gateway/camel_kura_router.md#managing-xml-camel-routes-using-web-ui)
with a need to restart your gateway or redeploy your quickstart bundle.