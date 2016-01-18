# Cloud Platform schema

Cloud Platform schema describes structure of the data consumed by platform [services](services/services.md).

Cloud Platform schema jar provides DTO objects representing schema. In order to use schema jar in your project include the following dependency in your POM
XML file:

    <dependency>
    	<groupId>io.rhiot</groupId>
    	<artifactId>rhiot-cloudplatform-schema</artifactId>
    	<version>${rhiot.version}</version>
    </dependency>

## Device schema

The device scheme represents the basic information about the device. Link Object provides basic information about the
metrics that device can provide.

    class Device {
    
        String deviceId;
    
        String registrationId;
    
        Date registrationDate;
    
        Date lastUpdate;
    
        String address;
    
        int port;
    
        InetSocketAddress registrationEndpointAddress;
    
        long lifeTimeInSec;
    
        String smsNumber;
    
        String lwM2mVersion;
    
        BindingMode bindingMode;
    
        List<LinkObject> objectLinks = new LinkedList<>();
    
        String rootPath;
        
    }
    
    class LinkObject {
    
        String url;
    
        Map<String, Object> attributes;
    
        Integer objectId;
    
        Integer objectInstanceId;
    
        Integer resourceId;
    
    }
