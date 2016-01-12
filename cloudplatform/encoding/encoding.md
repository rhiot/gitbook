# Payload encoding

It is important to stress that only client sending a payload to the Cloud Platform and service consuming message from
the IoT connector are aware of messages semantics. It basically means that it is client responsibility to properly
encode the message, while it is service responsibility to decode the message. Protocol adapters and IoT connector
are not aware of the message semantics. Please refer to the [Cloud Platform Architecture](../cloudplatform.md) for more
architectural details.

## Payload encoding in a PaaS environment

The default payload encoding used in Cload Platform PaaS environment is [JSON encoding](json.md). If you would like to
send a message to the PaaS protocol adapters or IoT connector encode those using [JSON encoding](json.md).

## Payload encoding SPI

In order to make encoding and decoding of the message easier Rhiot provides pluggable *Payload Encoding SPI*.

    import io.rhiot.cloudplatform.encoding.spi.PayloadEncoding;
    ...
    PayloadEncoding encoding = ...;
    Map<String, Object> payload = ...;
    byte[] message = encoding.encode(payload);
    send(message); // Sending serialized message to protocol adapter or IoT connector

Snippet below demonstrates how to use payload encoding SPI to decode a message read from IoT Connector or protocol
adapter:

    import io.rhiot.cloudplatform.encoding.spi.PayloadEncoding;
    ...
    PayloadEncoding encoding = ...;
    byte[] serializedPayload = ...; // Received serialized message from IoT Connector or protocol adapter
    Map<String, Object> payload = (Map<String, Object>) encoding.encode(serializedPayload);
    // Sending serialized message to protocol adapter or IoT connector

### Using payload encoding programatically in a Spring Boot runtime

In order to use `PayloadEncoding` instance programatically in a Spring Boot runtime, inject it into your component:

    import io.rhiot.cloudplatform.encoding.spi.PayloadEncoding;
    ...
    public class MyComponent {


        PayloadEncoding payloadEncoding;

        public MyComponent(PayloadEncoding payloadEncoding) {
            this.payloadEncoding = payloadEncoding;
        }

        public void operation() {
            byte[] encodedPayload = payloadEncoding.encode("mymessage");
           sendMessage(encodedPayload);
        }

        ...

    }
