# Camel Webcam component

Camel [Webcam](http://webcam-capture.sarxos.pl/) component can be used to capture still images and detect motion.
With Camel Webcam you can connect a camera to your device's USB port, or install the camera mod on the Raspberry Pi board for example,
and poll for an image periodically and respond to motion events.
The body of the message is the current image as a BufferedInputStream, while motion events are stored in the header 'io.rhiot.webcam.webcamMotionEvent'.
This event may be useful for getting the image captured prior to the motion event, as well the Points where the motion occurred and the center of motion gravity.


## Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-webcam</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

Avaliable for rhiot.version >= 0.1.2

#### URI format


The Camel endpoint URI format for the Webcam consumer is as follows:

    webcam:label

Where `label` can be replaced with any text label:

    from("webcam:spycam").
      to("file:///var/spycam");

This route creates a scheduled consumer taking 1 frame per second and writes it to file in the PNG format.

Alternatively, respond to motion events by setting the endpoint parameter 'motion' to true.

    from("webcam:spycam?motion=true").
      to("file:///var/spycam");

You can also poll the webcam for a single image, for example;

    from("jms:webcam").
      to("webcam:spycam");

Specify the resolution with custom width and height, or the resolution name;

    from("webcam:spycam?resolution=HD720").
      to("seda:cam")

#### Options

| Option                   | Default value                                                                 | Description   |
|:-------------------------|:-----------------------------------------------------------------------       |:------------- |
| `consumer.deviceName`    |                                                                               | Specify the webcam device name, if absent the first device available is used, eg /dev/video0.  |
| `consumer.format`        | PNG                                                                           | Capture format, one of 'PNG,GIF,JPG'  |
| `consumer.initialDelay`  | 1000                                                                          | Milliseconds before the polling starts. Applies only to scheduled consumers. |
| `consumer.motion`        | false                                                                         | Whether to listen for motion events.                                |
| `consumer.motionInterval`| 500                                                                           | Interval in milliseconds to detect motion.                               |
| `consumer.pixelThreshold`| 25                                                                            | Intensity threshold when motion is detected (0 - 255)  |
| `consumer.areaThreshold` | 0.2                                                                           | Percentage threshold of image covered by motion (0 - 100)  |
| `consumer.motionInertia` | -1 (2 * motionInterval)                                                       | Motion inertia (time when motion is valid)  |
| `consumer.resolution`    | QVGA                                                                          | Resolution to use (PAL, HD720, VGA etc)     |
| `consumer.width`         | 320                                                                           | Resolution width, note that resolution takes precendence.  |
| `consumer.height`        | 240                                                                           | Resolution height, note that resolution takes precendence.  |
| `consumer.delay`         | 1000                                                                          | Delay in milliseconds. Applies only to scheduled consumers.  |
| `consumer.useFixedDelay` | false | Set to true to use a fixed delay between polls, otherwise fixed rate is used. See ScheduledExecutorService in JDK for details. |


#### Process manager

Process manager is also used by the Webcam component to execute Linux commands responsible for starting the webcam driver and  
configuring the image format and resolution. If for some reason you would like to change
the default implementation of the process manager used by Camel (i.e. `io.rhiot.utils.process.DefaultProcessManager`),
you can set it on the component level:

    WebcamComponent webcam = new WebcamComponent();
    webcam.setProcessManager(new CustomProcessManager());
    camelContext.addComponent("webcam", webcam);

If custom process manager is not set on the component, Camel will try to find the manager instance in the
[registry](http://camel.apache.org/registry.html). So for example for Spring application, you can just configure
the manager as the bean:

    @Bean
    ProcessManager myProcessManager() {
        new CustomProcessManager();
    }

Custom process manager may be useful if your Linux distribution or camera requires executing some unusual commands
in order to load the v4l2 (Video for Linux) module.


#### Installer

For some Linux+webcam combinations, the webcam component requires `v4l-utils` and its dependencies to be installed on an
operating system. By default the Webcam component installs `v4l-utils` using apt-get, you can configure the installer or
set an alternate one on the component:

    WebcamComponent webcam = new WebcamComponent();
    webcam.setInstaller(new BrewInstaller());
    camelContext.addComponent("webcam", webcam);    

You can also specify alternate dependencies for your platform, if your platform uses my-custom-v4l-utils, configure the component as follows:

    WebcamComponent webcam = new WebcamComponent();
    webcam.setRequiredPackages("my-custom-v4l-utils");  //comma-separated list of packages to install
    camelContext.addComponent("webcam", webcam);    

If an Installer is not set on the component, Camel will try to find an instance in the
[registry](http://camel.apache.org/registry.html). So for example for Spring application, you can configure the installer as a bean:

    @Bean
    Installer myInstaller() {
        CustomInstaller installer = new CustomInstaller();
        installer.setTimeout(60000 * 10); //Allow up to 10 minutes to install packages
    }

By default an installer ignores problems with the webcam packages installation and only logs the warning using a
logger WARN message. If you would like the component to thrown an exception instead of logging a message, set
`ignoreInstallerProblems` property of the `WebcamComponent` to `true`:

    WebcamComponent webcam = new WebcamComponent();
    webcam.setIgnoreInstallerProblems(true);
    camelContext.addComponent("webcam", webcam);

