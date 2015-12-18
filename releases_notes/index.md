# Rhiot release notes

## 0.1.3 (2015-12-18)

New features:
- Migrated documentation to GitBook (https://rhiot.gitbooks.io/rhiotdocumentation/content/)
- Created #rhiot IRC channel for discussions :)
- Added support for Kura gateways
- Create RhiotKuraRouter
- Created Rhiot shell
- Created device-send command
- Created kura-config-ini command
- Provided “rhiot raspbian-install” command
- Added “append” option to device-config command
- Created camel-lwm2m component
- Created GPIO component for camel-kura
- Created Kura CloudService component
- Created PojosrKuraServer
- Created camel-framebuffer component
- Created camel-bluetooth component
- Created camel-openimaj component for face detection
- Created Kura Camel quickstart
- Add support for reading configuration from a thread local

Improvements:
- Don’t install docker if MacOSX for rhiot command
- Rhiot CMD should be using “DevAgent” background process
- Camel-kura shouldn’t depend of “org.eclipse.kura.linux.net”
- Made GPIOProducer more dynamic
- camel-webcam should install v4l-utils if it’s not already installed
- WebcamComponent should analyze modprobe result to ensure v4l2 module can be loaded
- GPSD component should make sure that GPSD is installed
- 
Bug fixes:
- Provided state and shutdownState for kura-gpio Endpoint
- GPIOProducer shouldn’t overwrite the “action” on each message
- WebcamComponent#installer setter and getters are missing
- WebCam component should not fail when non-root user is running it

## 0.1.2  (2015-10-22)

- Deprecated BU353 component on the behalf of the GPSD component [#232](https://github.com/rhiot/rhiot/issues/232)
- Added GPS client coordinates type converter [#213](https://github.com/rhiot/rhiot/issues/213)
- Fixed: BU353 returns "FileNotFoundException: /dev/ttyUSB0 (Device or resource busy)" [#210](https://github.com/rhiot/rhiot/issues/210)
- Rhiot now supports reading Spring Boot application.properties file [#226](https://github.com/rhiot/rhiot/issues/226)
- Renamed com.github.camellabs.iot.vertx.camel.GroovyCamelVerticle to io.rhiot.vertx.camel.GroovyCamelVerticle [#207](https://github.com/rhiot/rhiot/issues/207)
- Moved camel-kura component from com.github.camellabs to io.rhiot package [#195](https://github.com/rhiot/rhiot/issues/195)
- Device detection is performed in parallel [#218](https://github.com/rhiot/rhiot/issues/218)
- Added "scan" command to the deployer [#217](https://github.com/rhiot/rhiot/issues/217)
- Deployer now allows to specify customized fat jar [#216](https://github.com/rhiot/rhiot/issues/216)
- Deployer now downloads the same gateway version as deployer version [#215](https://github.com/rhiot/rhiot/issues/215)
- Deployer now detects devices from multiple interfaces [#214](https://github.com/rhiot/rhiot/issues/214)
- Deployer now scans OSX network interfaces [#202](https://github.com/rhiot/rhiot/issues/202)
- Device Cloudlet MongoDB connection now timeouts faster [#191](https://github.com/rhiot/rhiot/issues/191)
- Add [Webcam camel component](https://github.com/rhiot/rhiot/issues/239), thx [@levackt](https://github.com/levackt) [#238](https://github.com/rhiot/rhiot/issues/239)


## 0.1.1  (2015-09-15)

- Changed project name from *Camel Labs* to *Rhiot*
- Added Dockerized Rhiot Cloud [#160](https://github.com/rhiot/rhiot/issues/160).
- Added device management cloudlet [#114](https://github.com/rhiot/rhiot/issues/114).
- Added web UI (aka Cloudlet Console [#129](https://github.com/rhiot/rhiot/issues/129).
- Created [Rhiot Cloud demo site](http://rhiot.net) [#155](https://github.com/rhiot/rhiot/issues/155).
- Added camel-gps-bu353 component [#93](https://github.com/rhiot/rhiot/issues/93).
- Migrated Gateway core from Spring Boot to Vert.x [#141](https://github.com/rhiot/rhiot/issues/141)  [#120](https://github.com/rhiot/rhiot/issues/120).
- Gateway should start Jolokia REST API on port 8778, not 8080 [#143](https://github.com/rhiot/rhiot/issues/143).
- Releasing Gateway core and application artifacts separately [#126](https://github.com/rhiot/rhiot/issues/126).
- Renamed `camellabs.iot.gateway.heartbeat.rate property` to `camellabs_iot_gateway_heartbeat_rate` [#124](https://github.com/rhiot/rhiot/issues/124).
- Gateway deletes logs after N megabytes limit is exceeded [#104](https://github.com/rhiot/rhiot/issues/104)  [#95](https://github.com/rhiot/rhiot/issues/95).
- Gateway reads properties from the `/etc/default/camel-labs-iot-gateway` [#98](https://github.com/rhiot/rhiot/issues/98).

## 0.1.0  (2015-06-02)

- Initial version of the Camel based [IoT field gateway](https://github.com/rhiot/rhiot/tree/master/iot#camel-iot-gateway)
- Raspbian [installation scripts](https://github.com/rhiot/rhiot/tree/master/iot#installing-gateway-on-the-raspbian) for the Camel IoT Gateway
- [Heartbeats support](https://github.com/rhiot/rhiot/tree/master/iot#device-heartbeats) for the Camel IoT Gateway (including logging, MQTT and LED signals)
- [Camel Kura WiFi component](https://github.com/rhiot/rhiot/tree/master/iot#camel-kura-wifi-component)
- [Camel Tinkerforge component](https://github.com/rhiot/rhiot/tree/master/iot#camel-tinkerforge-component)
- [Camel Pi4j component](https://github.com/rhiot/rhiot/tree/master/iot#camel-pi4j-component)
- [Camel PubNub component](https://github.com/rhiot/rhiot/tree/master/iot#camel-pubnub-component)
- Alpha version of the [Cloudlets](https://github.com/rhiot/rhiot/tree/master/iot#cloudlets) backend services, currently including [Geofencing Cloudlet](https://github.com/rhiot/rhiot/tree/master/iot/cloudlet/geofencing) and [Hawt.io-based UI plugin for it](https://github.com/rhiot/rhiot/tree/master/iot/cloudlet/geofencing)
