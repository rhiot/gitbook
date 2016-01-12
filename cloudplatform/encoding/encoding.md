# Payload encoding

It is important to stress that only client sending a payload to the Cloud Platform and service consuming message from
the IoT connector are aware of messages semantics. It basically means that it is client responsibility to properly
encode the message, while it is service responsibility to decode the message. Protocol adapters and IoT connector
are not aware of the message semantics. Please refer to the [Cloud Platform Architecture](../cloudplatform.md) for more
architectural details.

## Payload encoding SPI

In order to make encoding and decoding of the message easier Rhiot provides pluggable *Payload Encoding SPI*.

    PayloadEncoding encoding = ...;
    Map<String, Object> payload = ...;
    byte[] message = encoding.encode(payload);
    send(message); // Sending serialized message to protocol adapter or IoT connector



    PayloadEncoding encoding = ...;
    byte[] serializedPayload = ...; // Received serialized message from IoT connector or protocol adapter
    Map<String, Object> payload = (Map<String, Object>) encoding.encode(serializedPayload);
    // Sending serialized message to protocol adapter or IoT connector

## Payload encoding in PaaS environment

The default payload encoding used in Cload Platform PaaS environment is [JSON encoding](json.md).