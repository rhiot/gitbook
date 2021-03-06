# Rhiot command line tool (cmd)

The basic tool for Rhiot is the `rhiot` Bash command. In order to install Rhiot command, execute the following command:

    bash <(curl -sL https://goo.gl/z7RPEv)

In order to display all available commands with their options, execute the `rhiot` command with `--help` or `-h` option:

    rhiot --help

Rhiot command line assumes that there is Docker server running on your local machine. An actual commands execution is delegated by Docker client to the Rhiot Docker image. If Docker is not installed (or is installed in a version lower than a minimal version supported), Rhiot cmd will automatically install or upgrade Docker.

## Before you start

If you are using Raspbian, please be sure that the SSH account you are using to manage your device has root privileges.
In particular keep in mind that the default `pi` user provided by Raspbian doesn't have root privileges.

The easiest way to use root-enabled SSH user is to enable root login at your device. In order to do it, add `PermitRootLogin yes`
option to the `/etc/ssh/sshd_config` file:

    $ sudo nano /etc/ssh/sshd_config 
    # Authentication:
    LoginGraceTime 120
    #PermitRootLogin without-password
    PermitRootLogin yes
    StrictModes yes
    
Then restart your SSHD server:

    sudo service sshd restart
    
If you would like to authenticate yourself as root using password, you have to change it (as a default root password is
not set):

    sudo passwd root

For Raspbian, we recommend to use the default password - `raspberry`.

## device-config

This command allows to edit a configuration file on a remote device. The syntax of a command looks as follows:

    device-config file property value

For example:

    $ device-config /etc/config.txt delay 1000
    Updated /etc/config.txt - set property 'delay' to value '1000'.

Configuration file will be created if it doesn't exists.

SSH options:

* `--host` (`-ho`) host    Address of the target device. The device will be automatically detected if this option is not specified.
* `--port` (`-p`)          Port of the SSH server on a target device. Defaults to 22.
* `--username` (`-u`)      SSH username of the device. Defaults to 'root'.
* `--password` (`-pa`)     SSH password of the device. Defaults to 'raspberry'.

Other options:

*  `--append` (`-a`)       Appends to the given property instead of overriding it.

## device-scan

**Available since Rhiot 0.1.3.**  

To perform port scanning in your local network and display detected devices, execute a `rhiot device-scan` command:

    $ rhiot device-scan
    Scanning local networks for devices...

    ======================================
    Device type		IPv4 address
    --------------------------------------
    RaspberryPi2		/192.168.1.100

## device-send

**Available since Rhiot 0.1.3.**  

Sends file to a remote device. For example:

    device-send /etc/localfile /etc/remotefile

All the device-config command SSH options are also available for this command.

**Available since Rhiot 0.1.4.**

Using URL as a file source is accepted as well. For example to read file from HTTP and send it to a remote device:

    device-send http://example.com/file /etc/remotefile

You can also transfer Maven artifacts. For example to copy Google Guava jar to a remote device, execute the following
command:

    device-send mvn:com.google.guava/guava/18.0 /opt/guava.jar

## raspbian-config-boot

**Available since Rhiot 0.1.3.**  

Sets property on a `/boot/config.txt` file on a remote device. All the [device-config](#deviceconfig) command options 
are also available for this command.

Syntax:

    raspbian-config-boot property value

Example:
    
    raspbian-config-boot hdmi_drive 2

## kura-config-bootdelegation

Syntax: `kura-config-bootdelegation`

Example: `kura-config-bootdelegation`

Enables OSGi boot delegation for Sun packages i.e. adds `org.osgi.framework.bootdelegation=sun.*,com.sun.*` property to
the `/opt/eclipse/kura/kura/config.ini` file on a remote device. This settings is required to use libraries relying
on `com.sun` and `sun` packages deployed into Kura.

All the [device-config](#deviceconfig) command options (like `--address` or `--port` for specifying device address and 
SSH port) are also available for this command.

## kura-config-ini

**Available since Rhiot 0.1.3.**  

Sets property on a `/opt/eclipse/kura/kura/config.ini` file on a remote device. All the [device-config](#deviceconfig) command options 
are also available for this command.

Syntax:

    kura-config-ini property value

Example:
    
    kura-config-ini equinox.ds.debug true

## kura-install

**Available since Rhiot 0.1.4.**

Syntax: kura-install

Example: kura-install

Installs Kura Server (1.3.0 with Web UI) into your target device, together with all dependant packages needed.

All the `device-config` command options are also available for this command.

## kura-install-felix-fileinstall

**Available since Rhiot 0.1.4.**  

Syntax: kura-install-felix-fileinstall

Example: kura-install-felix-fileinstall

Deploys [Felix File Install](http://felix.apache.org/documentation/subprojects/apache-felix-file-install.html) into your
Kura server. After Felix File Install is installed into your server, you can drop
bundle jars into `/opt/eclipse/kura/deploy` in order to automatically deploy those.

All the `device-config` command options are also available for this command.

## raspbian-install

Installs Raspbian Jessie to a given SD card. For example to install Raspbian to SD card device `/dev/sdd1`, execute the
following command:

    rhiot raspbian-install ssd1
    
Rhiot will download a Raspbian image for you (if needed), extract it and install to the target SD card.

## deploy-gateway

Deploys gateway to a detected device. In order to install Rhiot gateway on a target device connect it to your local
network (using WiFi or the ethernet cable). Then execute the following command on the laptop connected to the same network as your device

    rhiot deploy-gateway

From this point forward Rhiot gateway will be installed on your device as `rhiot-gateway` service and started
whenever the device boots up. Under the hood, gateway deployer performs a simple port scanning in a local network
and attempts to connect to supported devices using the default SSH credentials.

To see all the options available for the gateway deployer, execute the following command:

    rhiot --help

In case of problems with the gateway, you can try to run it in verbose mode (called *debug mode*):

    rhiot deploy-gateway --debug

You can also configure the gateway during the deployment process using the `-P` option. For example to set the configuration property
responsible for gateway heartbeats interval, execute the following command:

    rhiot deploy-gateway -Pcamellabs_iot_gateway_heartbeat_rate=10000

You can use the `-P` option multiple times:

    rhiot deploy-gateway -Pfoo=bar -Pbar=qux

If you would like to use different SSH credentials than default (username `pi`, password `raspberry`), then pass the
`--username` and `--password` options to the deployer:

    rhiot deploy-gateway --username=john --password=secret

If you would like to deploy your customized gateway fat jar, you can specify its Maven coordinates when running the deployer:

    rhiot deploy-gateway --artifact=com.example:custom-gateway:1.0
