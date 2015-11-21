# Camel Framebuffer component

Camel Framebuffer component can be used to manage any Linux Framebuffer.
This component should work with any /dev/fb* linux device.

## Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-framebuffer</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

 Avaliable for rhiot.version >= 0.1.3


## URI format for Framebuffer

    framebuffer://name

*name* can /dev/*fbN* or /sys/class/graphics/name content file

## Optional URI Parameters

| Parameter            | Default value        | Description   |
|----------------------|----------------------|---------------|
| `name`               |                      |               |

### Producing:

Route that consumes messages from mychannel:

    from("direct:start").
    .to("framebuffer://RPi-Sense FB");

If the body message is empty framebuffer component copies the framebuffer file to the In Body Exchange as byte[].
If the body message not empty framebuffer component copies the In Body Exchange (byte[]) directly to the framebuffer file.
