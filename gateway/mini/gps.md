# GPS in the gateway

Rhiot gateway comes with a dedicated GPS support. It makes it easier to collect, store and forward the GPS data to a
data center (to the [Rhiot Cloud](../../backend/index.md) in particular).
Under the hood gateway GPS support uses the [GPSD component](https://github.com/rhiot/rhiot/blob/master/docs/readme.md#camel-gpsd-component).

#### Enabling GPS receiver

In order to start collecting GPS data on the gateway device, just add `gps=true` property to the gateway configuration.
Gateway with the GPS support enabled will poll the GPS receiver every 5 seconds and store the GPS coordinates in the
`/var/rhiot/gps` directory (or the other directory indicated using `gps_store_directory` configuration property). The data
collected from the GPS receiver will be serialized to the JSON format. Each GPS read is saved to in a dedicated file.

The JSON schema of the collected coordinates is similar to teh following example:

    {"timestamp": 1445356703704,
      "lat": 20.0, "lng": 30.0}

#### Enriching GPS data

Sometimes you would like to collect not only the GPS coordinates, but some other sensor data associated with the given
location. For example to track the temperature or WiFi networks available in various locations. If you would like to
enrich collected GPS data with value read from some other endpoint, set the `gps_enrich` property to the Camel endpoint
URI value. For example setting `gps_enrich=kura-wifi:*/*` can be used to poll for available WiFi networks using
[Camel Kura WiFi component](https://github.com/rhiot/rhiot/blob/master/docs/readme.md#camel-kura-wifi-component) and adding
the results to collected GPS payloads.

The enriched JSON schema of the collected coordinates is similar to teh following example:

    {"timestamp": 1445356703704,
      "lat": 20.0, "lng": 30.0,
      "enriched": {
        "foo": "bar"
      }
    }      

#### Sending collected GPS data with a data center

If you would like to synchronize the collected GPS data with a data center, consider using the out-of-the-box
functionality that Rhiot gateway offers. In order to enable an automatic synchronization of the GPS coordinates, set the
`gps_cloudlet_sync` configuration property to `true`.

The gateway synchronization route reads GPS messages saved in the
gateway store (see the section above) and sends those to the destination endpoint. The target Camel endpoint can be specified using
the `gps_cloudlet_endpoint` configuration property. For example the following property tells a gateway to send
the GPS data to a MQTT broker:

    gps_cloudlet_endpoint=paho:topic?brokerUrl=tcp://broker.com:1234

The default data center endpoint is Netty REST call (using POST method) - `netty4-http:http://${cloudletAddress}/api/document/save/GpsCoordinates`, where
`gps_cloudlet_address` configuration property has to be configured to match your HTTP API address (for example
`gps_cloudlet_address=myapi.com:8080`).

The default data format used by the gateway is JSON, where the message schema is similar to the folowwing example:

     {"client": "my-device",
       "clientId": "local-id-generated-on-client",
       "timestamp": 1445356703704,
       "latitude": 20.0,
       "longitude": 30.0,
       "enriched": {
         "foo": "bar"
       }
     }
