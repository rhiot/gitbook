# Service binding

Service binding is a general purpose backend service which can be used to bind communication coming through IoT
connector to the POJO service bean. Let's take the following service interface as an example:

    public interface TestService {

        int count(int number);

        int sizeOf(Map map);

        int numberPlusSizeOf(int number, Map map);

        int numberPlusNumberPlusSizeOf(int number1, int number2, Map map);

    }

And some implementation of it:

    @Component("test")
    public class TestInterfaceImpl implements TestService {

        @Override
        public int count(int number) {
            return number;
        }

        @Override
        public int sizeOf(Map map) {
            return map.size();
        }

        @Override
        public int numberPlusSizeOf(int number, Map map) {
            return number + map.size();
        }

        @Override
        public int numberPlusNumberPlusSizeOf(int number1, int number2, Map map) {
            return number1 + number2 + map.size();
        }

    }

Now let's register the `ServiceBinding` route in your backend service:

    import io.rhiot.cloudplatform.service.binding.ServiceBinding;

    ...

    @Bean
    ServiceBinding testServiceBinding(PayloadEncoding payloadEncoding) {
        return new ServiceBinding(payloadEncoding, "test");
    }

The service binding above allows us to bind all messages sent to the `test.>` IoT Connector AMQP channel to the service
we registered as `test`.

## Binding convention

Service binding relies on a convention-over-configuration to match IoT Connector traffic with given service. For the
example above all messages matching the destination pattern `test.OPERATION.ARG1.ARG2.ARG3...` are bound the the
appropriate method invocation. For example:

    QUEUE: test.count.10
        =>
    TestInterfaceImpl.count(10)

The last argument of operation can be also sent as a message body. For example:

    QUEUE: test.count
    BODY: 10
        =>
    TestInterfaceImpl.count(10)

    QUEUE: test.sizeOf
    BODY: {"payload: {"foo": "bar"}"} // Assuming that default JSON payload encoding is used
        =>
    TestInterfaceImpl.sizeOf([foo: "bar"])

    QUEUE: test.numberPlusSizeOf.10
    BODY: {"payload: {"foo": "bar"}"}
        =>
    TestInterfaceImpl.numberPlusSizeOf(10, [foo: "bar"])

### Binding headers to operation arguments

Also binding algorithm looks for message headers following this convention:

    RHIOT_ARG0, RHIOT_ARG1, RHIOT_ARG2, ...

The values of those headers are bound to the service invocation as arguments:

    QUEUE: test.count
    HEADERS: RHIOT_ARG0=10
        =>
    TestInterfaceImpl.count(10)

Keep in mind that channel arguments take precedence over header arguments:

    QUEUE: test.numberPlusNumberPlusSizeOf.10
    HEADERS: RHIOT_ARG0=20
    BODY: {"payload: {"foo": "bar"}"}
        =>
    TestInterfaceImpl.numberPlusNumberPlusSizeOf(10, 20, [foo: "bar"])

## Using service binding programatically in Spring Boot runtime

In order to use service binding routes in your Spring Boot application add the following jar to your classpath:

	<dependency>
		<groupId>io.rhiot</groupId>
		<artifactId>rhiot-cloudplatform-service-binding</artifactId>
		<version>${rhiot.version}</version>
	</dependency>

Then register the service bean you want to invoke:

    @Component("myService")
    public class MyService {
        ...
    }

Register the binding for the given bean:

    import io.rhiot.cloudplatform.service.binding.ServiceBinding;

    ...

    @Bean
    ServiceBinding testServiceBinding(PayloadEncoding payloadEncoding) {
        return new ServiceBinding(payloadEncoding, "myService");
    }

Finally start your platform:

    import io.rhiot.cloudplatform.runtime.spring.CloudPlatform
    ...
    new CloudPlatform().start();

As soon as platform is started, IoT Connector destination `myService.>` will be bound to the `MyService` bean from your
Spring context.