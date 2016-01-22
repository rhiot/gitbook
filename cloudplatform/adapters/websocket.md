# Websocket protocol adapter

Websocket protocol adapter bridges Websocket requests with the ActiveMQ Transport. It therefore allows you to expose your services via Websocket API.


### Starting REST protocol adapter programatically in Spring runtime

In the opposite way of the other protocol adapters, Websocket protocol adapter comes from ActiveMQ broker, just need the following jar into your POM file and set several system wide parameters

    	<dependency>
    		<groupId>io.rhiot</groupId>
    		<artifactId>rhiot-cloudplatform-runtime-spring</artifactId>
    		<version>${rhiot.version}</version>
    	</dependency>

Then connect to the Cloud Platform:

        import io.rhiot.cloudplatform.runtime.spring.CloudPlatform
        ...
        System.setProperty("spring.activemq.broker.websocketEnabled", "true");
        System.setProperty("spring.activemq.broker.websocketPort", "9090" );
        new CloudPlatform().start();


## Spring runtime Configuration

By default Websocket connector listens on HTTP port 9090. You can change this port by setting `spring.activemq.broker.websocketPort` property.