# Cloud Platform quickstart

The Cloud Platform quickstart is useful to understand how the Cloud Platform runtime and all related components like protocol adapters and services can be used to develop a complete IoT end to end solution based on the IoT Connector architecture. It shows how it's possible to send data from clients (devices) to the backend system using a protocol adapter like the HTTP REST and acquire this data with a custom service. Of course, the same scenario is possible using IoT Connector directly without any kind of adapter layer.

# The simple scenario

This quickstart relies on a simple scenario with a device which is able to get temperature from the external environment and send it to a custom service in a Telemetry fashion. The device is also able to execute a query against the service to get the temperature threshold value for raising alarm.

# Enabling the Cloud Platform programmatically

First of all, we have to enable the Cloud Platform runtime programmatically so the related dependency is needed :

```
<dependency>
	<groupId>io.rhiot</groupId>
  	<artifactId>rhiot-cloudplatform-runtime-spring</artifactId>
</dependency>
```

The quickstart is made of a group of tests and the main *RhiotCloudPlatformTest* class extends the *CloudPlatformTest* base class which is able to start the Cloud Platform runtime and the underneath Active MQ broker for communication purpose. The Active MQ broker is needed because it's the core of the IoT Connector infrastructure, so we need its dependency in our project.

```
<dependency>
	<groupId>io.rhiot</groupId>
	<artifactId>rhiot-spring-activemq</artifactId>
	<scope>test</scope>
</dependency>

```

For more information you can find them [here](https://rhiot.gitbooks.io/rhiotdocumentation/content/cloudplatform/starting.html).

# The custom Temperature Service

The custom service is implemented through the following *TemperatureService* interface :

```java
public static interface TemperatureService {
		
	static final String CHANNEL = "temperature";
		
	Boolean store(String deviceId, double temperature);
		
	double threshold(String deviceId);
}

```

The *store* method is used for Telemetry purpose and a device can call it to send temperature value to the cloud platform. The *threshold* method is used for Enquiry purpose so that a device can query the backend system about the temperature threshold it has to use to raise alarm for example. Of course, this interface needs to be implemented by a concrete class like the *TemperatureServiceImpl* one in the example.

```java
@Component(TemperatureService.CHANNEL)
public static class TemperatureServiceImpl implements TemperatureService {
		
	public Boolean store(String deviceId, double temperature) {
		LOG.info("Storing temperature " + temperature + " for device " +  deviceId);
		return true;
	}

	public double threshold(String deviceId) {
		LOG.info("Temperature threshold for device " + deviceId);
		return 25.0;
	}
}

```

After implementing the custom service (exposed as a Spring component), we need to bind it inside the cloud platform and assign a channel used for IoT Connector communications.

```java
@Bean
ServiceBinding temperatureServiceBinding(PayloadEncoding payloadEncoding) {
	return new ServiceBinding(payloadEncoding, TemperatureService.CHANNEL);
}
```

In order to use the service binding, the related component is needed as dependency.

```
<dependency>
	<groupId>io.rhiot</groupId>
	<artifactId>rhiot-cloudplatform-service-binding</artifactId>
	<scope>test</scope>
</dependency>
```

# Encoding payload

The cloud platform provides the possibility for the user to choose how the messages payload should be encoded. It's possible thanks to the [payload encoding](https://rhiot.gitbooks.io/rhiotdocumentation/content/cloudplatform/encoding/encoding.html) feature. The current quickstart relies on the [JSON payload encoding](https://rhiot.gitbooks.io/rhiotdocumentation/content/cloudplatform/encoding/json.html) implementation.

```
<dependency>
	<groupId>io.rhiot</groupId>
  	<artifactId>rhiot-cloudplatform-encoding-json</artifactId>
</dependency>
```

# Using HTTP REST adapter

In order to send and get data from device using the standard HTTP protocol, the related [HTTP REST protocol adapter](https://rhiot.gitbooks.io/rhiotdocumentation/content/cloudplatform/adapters/rest.html) is needed inside the cloud platform because HTTP requests (and replies) needs to be adapted to/from IoT Connector shape.

```
<dependency>
	<groupId>io.rhiot</groupId>
	<artifactId>rhiot-cloudplatform-adapter-rest</artifactId>
	<scope>test</scope>
</dependency>
```

For storing a temperature value, using the HTTP protocol adapter, first of all we need to identify the URL for the service which has the current format :

```
http://localhost:8080/[service]/[operation]
```

where there is the channel "service" name and the "operation" to execute ("store" and "threshold" in this scenario).
Regarding the store operation, the two parameters related to deviceId and temperature value can be put inside the HTTP request in two ways.

The first one is based on HTTP headers with the RHIOT_ARG prefix and this is the preferred one to use.

```java

@Test
public void httpRestStoreTemperatureHeader() {
	
	double temperature = getTemperature(); 
	
	// HTTP REST URL format : http://localhost:8080/[service]/[operation]
	String url = String.format("http://localhost:8080/%s/store", TemperatureService.CHANNEL);
	
	// put parameters (device ID and temperature value) inside RHIOT prefixed HTTP headers
	HttpHeaders headers = new HttpHeaders();
	headers.set("RHIOT_ARG0", DEVICE_ID);
	headers.set("RHIOT_ARG1", String.valueOf(temperature));
	
	ResponseEntity<Map> response = rest.exchange(url, HttpMethod.POST, new HttpEntity<byte[]>(headers), Map.class);
	
	boolean stored = (Boolean)response.getBody().get("payload");
	LOG.info("Temperature stored " + stored);
	
	Truth.assertThat(stored).isTrue();
}
```

The second way is based on passing one parameter (the deviceId) in the URL and the second one (the temperature value) in the HTTP body. In this second scenario, the service method changes in :

```
http://localhost:8080/[service]/[operation]/[parameter]
```

```java
@Test
public void httpRestStoreTemperatureBody() {
	
	double temperature = getTemperature();
	
	// HTTP REST URL format : http://localhost:8080/[service]/[operation]/[parameter]
	// put device ID in the URL path
	String url = String.format("http://localhost:8080/%s/store/%s", TemperatureService.CHANNEL, DEVICE_ID);
	
	// encode temperature ...
	byte[] request = payloadEncoding.encode(temperature);
	// ... and put its value inside the HTTP request body
	Map response = rest.postForObject(url, request, Map.class);
	
	boolean stored = (Boolean)response.get("payload");
	LOG.info("Temperature stored " + stored);
	
	Truth.assertThat(stored).isTrue();
}
```

In order to get the temperature threshold from the service, the good way is to put the only deviceId parameter inside the HTTP request as an header prefixed by RHIOT_ARG.

```
@Test
public void httpRestTemperatureThreshold() {
	
	// put parameter device ID inside RHIOT prefixed HTTP headers
	HttpHeaders headers = new HttpHeaders();
	headers.set("RHIOT_ARG0", DEVICE_ID);
	
	// HTTP REST URL format : http://localhost:8080/[service]/[operation]
	String url = String.format("http://localhost:8080/%s/threshold", TemperatureService.CHANNEL);
	
	ResponseEntity<Map> response = rest.exchange(url, HttpMethod.GET, new HttpEntity<byte[]>(headers), Map.class);
	
	double threshold = (Double)response.getBody().get("payload");
	LOG.info("Temperature threshold " + threshold);
	
	Truth.assertThat(threshold).isNotNull();
}
```

In all above cases, the platform returns the reply encoded in JSON format with a "payload" field filled with data response.

# Using the IoT Connector

Another way to connect to the cloud platform is to use the IoT Connector component natively. In this case we can use the communication channel formatted in the following way :

```
[service].[operation]
```

with all related parameters passed along as fields inside an header structure. With the IoT Connector component, it's possible to send/
receive to/from the communication bus in a fire and forget manner or request/reply way.

```
@Test
public void iotConnectorStoreTemperature() {
	
	double temperature = getTemperature();
	
	// IoT connector channel format [service].[operation]
	String channel = String.format("%s.%s", TemperatureService.CHANNEL, "store");
	
	// put parameters (device ID and temperature value) as headers
	boolean stored = connector.fromBus(channel, boolean.class, Header.arguments(DEVICE_ID, temperature));
	LOG.info("Temperature stored " + stored);
	
	Truth.assertThat(stored).isTrue();
}

@Test
public void iotConnectorTemperatureThreshold() {
	
	// IoT connector channel format [service].[operation]
	String channel = String.format("%s.%s", TemperatureService.CHANNEL, "threshold");
	
	double threshold = connector.fromBus(channel, double.class, Header.arguments(DEVICE_ID));
	LOG.info("Temperature threshold " + threshold);
	
	Truth.assertThat(threshold).isNotNull();
}

```

In these cases no JSON payload encoding is involved and the method invocation returns a strongly typed data (storing result and threshold temperature).






