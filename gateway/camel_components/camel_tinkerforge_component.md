#Camel TinkerForge component

The Camel Tinkerforge component can be used to connect to the TinkerForge brick deamon.

## Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-tinkerforge</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

 Avaliable for rhiot.version >= 0.1.1

## General URI format

    tinkerforge:/<brickletType>/<uid>[?parameter=value][&parameter=value]

By default a connection is created to the brickd process running on localhost using no authentication.
If you want to connect to another host use the following format:

    tinkerforge://[username:password@]<host>[:port]/<brickletType>/<uid>[?parameter=value][&parameter=value]

The following values are currently supported as brickletType:

* ambientlight
* temperature
* lcd20x4
* humidity
* io4
* io16
* distance
* ledstrip
* motion
* soundintensity
* piezospeaker
* linearpoti
* rotarypoti
* dualrelay
* solidstaterelay

### Ambientlight

    from("tinkerforge:/ambientlight/al1")
    .to("log:default");

### Temperature

    from("tinkerforge:/temperature/T1")
    .to("log:default");

### Lcd20x4

The LCD 20x4 bricklet has a character based screen that can display 20 characters on 4 rows.

#### Optional URI Parameters

| Parameter | Default value | Description                              |
|-----------|---------------|------------------------------------------|
| line      | 0             | Show message on line 0                   |
| position  | 0             | Show message starting at position 0      |

    from("tinkerforge:/temperature/T1
    .to("tinkerforge:/lcd20x4/lcd1?line=2&position=10

The parameters can be overridden for individual messages by settings them as headers on the exchange:

    from("tinkerforge:/temperature/T1
    .setHeader("line", constant("2"))
    .setHeader("position", constant("10"))
    .to("tinkerforge:/lcd20x4/lcd1");

### Humidity

     from("tinkerforge:/humidity/H1")
     .to("log:default");

### Io16

The IO16 bricklet has 2 ports (A and B) which both have 8 IO pins. Consuming and producing
messages happens on port level. So only the port can be specified in the URI and the pin will
be a header on the exchange.

#### Consuming:

    from("tinkerforge:/io16/io9?ioport=a")
    .to("log:default?showHeaders=true");

#### Producing

    from("timer:default?period=2000")
    .setHeader("iopin", constant(0))
    .setHeader("duration", constant(1000))
    .setBody(constant("on"))
    .to("tinkerforge:/io16/io9?ioport=b");

