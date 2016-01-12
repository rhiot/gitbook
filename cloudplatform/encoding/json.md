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