# Rhiot command line tool (cmd)

The basic tool for Rhiot is the `rhiot` Bash command. In order to install Rhiot command, execute the following command:

    bash <(curl -sL https://goo.gl/z7RPEv)

In order to display all available commands with their options, execute the `rhiot` command with `--help` or `-h` option:

    rhiot --help

Rhiot command line assumes that there is Docker server running on your local machine. An actual commands execution is delegated by Docker client to the Rhiot Docker image. If Docker is not installed (or is installed in a version lower than a minimal version supported), Rhiot cmd will automatically install or upgrade Docker.

## shell-start

Starts background shell process (if one is not started already). You can connect to a [shell](shell.md) process using SSH and
default credentials:

    ssh rhiot@localhost -p 2000
    password: rhiot

## rhiot raspbian-install

Installs Raspbian Jessie to a given SD card. For example to install Raspbian to SD card device `/dev/sdd1`, execute the
following command:

    rhiot raspbian-install ssd1
    
Rhiot will download a Raspbian image for you (if needed), extract it and install to the target SD card.

## rhiot scan

**Available since Rhiot 0.1.3.**  

To perform port scanning in your local network and display detected devices, execute a `rhiot scan` command:

    $ rhiot scan
    Scanning local networks for devices...

    ======================================
    Device type		IPv4 address
    --------------------------------------
    RaspberryPi2		/192.168.1.100

## rhiot deploy-gateway

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
