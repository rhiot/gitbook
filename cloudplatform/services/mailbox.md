# Mailbox service

Mailbox service can be used to send messages to the devices. Each device has a server-side mailbox associated with it.
If a message is put into into a device outbox, the former will be sent to a inbox of a destination device.

## Mailbox API

If you would like to send a message to a device identified using ID `targetDevice`, send that message to the following
channel:

    BODY:       Map<String, Object> mail
    HEADERS:    RHIOT_ARG_0 = targetDevice
    CHANNEL:    mailbox.outbox

If you would like to subscribe to an inbox of a device identified as `targetDevice`, subscribe to the following channel:

    CHANNEL:    inbox.targetDevice

## Mail store

Each mail piece sent to the mailbox service is processed by `io.rhiot.cloudplatform.service.mailbox.MailStore` component when picked up from the outbox and
before it is sent to a target inbox. Mail store provide a way to track the history of the mailing passed through the
platform.

    public interface MailStore {

        void save(String receiver, Map<String, Object> mail);

    }

### InMemoryMailStore

The `io.rhiot.cloudplatform.service.mailbox.InMemoryMailStore` is an implementation of mail store which keeps last
message for each device in a memory. Keep in mind that `InMemoryMailStore` is not intended to be used in a production
environment.

    InMemoryMailStore store = new InMemoryMailStore();
    Map<String, Object> mail = new HashMap<>();
    mail.put("subject", "test");
    mail.put("body", "Hello world!");
    store.save("deviceId", mail);
    Map<String, Map<String, Object>> allMails = store.mails();

## Running mailbox service in Spring Boot runtime

In order to use mailbox service in your Spring Boot application add the following jar to your classpath:

	<dependency>
		<groupId>io.rhiot</groupId>
		<artifactId>rhiot-cloudplatform-service-mailbox</artifactId>
		<version>${rhiot.version}</version>
	</dependency>

Then start your platform:

    import io.rhiot.cloudplatform.runtime.spring.CloudPlatform
    ...
    new CloudPlatform().start();

As soon as platform is started mailbox service will be detected an initialized.

### Configuring mail store in Spring Boot environment

If you would like to change default `InMemoryMailStore` implementation of mail store to something production-ready,
add bean implementing `MailStore` interface to your application context:

    import io.rhiot.cloudplatform.service.mailbox.MailStore
    ...
    @Bean
    MailStore mailStore() {
        return new MyMailStore();
    }
