# JSON payload encoding

JSON payload encoding serializes message object into JSON with root element named `payload`:

    import io.rhiot.cloudplatform.encoding.json.JsonPayloadEncoding;

    ...

    new JsonPayloadEncoding().encode("foo");    =>  {"payload": "foo"}

    new JsonPayloadEncoding().encode(100);    =>  {"payload": 100}

    Map<String, Object> map = new HashMap<>();
    map.put("foo", "bar");
    new JsonPayloadEncoding().encode(map);    =>  {"payload": {"foo": "bar"}}

    new JsonPayloadEncoding().encode(new Person().name("Henry"));    =>  {"payload": {"name": "Henry"}}

The `payload` wrapper is dropped when decoding the payload:

    import io.rhiot.cloudplatform.encoding.json.JsonPayloadEncoding;
    ...
    byte[] payload = "{\"payload\": \"foo\"}".getBytes();
    new JsonPayloadEncoding().decode(payload);    =>  "foo"

Under the hood JSON encoding uses [Jackson](http://wiki.fasterxml.com/JacksonHome) library.

## Using JSON payload encoding in PaaS environment

JON payload is a default payload used in a Cload Platform PaaS environment.  If you would like to send a message to the
PaaS protocol adapters or IoT connector encode those using [JSON encoding](json.md).

## Using JSON payload programatically in a Spring Boot runtime

In order to enable JSON encoing in your Cloud Platform application, just add the following jar into your POM file.

    <dependency>
    	<groupId>io.rhiot</groupId>
    	<artifactId>rhiot-cloudplatform-encoding-json</artifactId>
    	<version>${rhiot.version}</version>
    </dependency>

Spring Boot runtime automatically detects and starts REST protocol adapter as soon `CloudPlatform` instance is started:

    new CloudPlatform().start();

