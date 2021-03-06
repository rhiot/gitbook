# Camel Kura deploy manager component

Deploy manager can be used to manage bundles installed on gateway devices using various channels (like MQTT topics). This feature is particularly
useful for gateway devices behind firewalls or NAT which cannot be managed directly using SSH.

## Installing bundles

In order to install bundles to the gateway device via MQTT topic, deploy the following route into your gateway device:

    import io.rhiot.component.kura.deploymanager.InstallProcessor;

    ...

    from("paho:deploymanager/install/+").
        setHeader("DeployManager.topic").header("CamelMqttTopic").
        process(new InstallProcessor());

The consumer part of the route can be any Camel component, as long as `DeployManager.topic` header required by `InstallProcessor`
is properly set. `InstallProcessor` reads the last segment of the `DeployManager.topic` header and uses it as a name of
the bundle to be installed. The binary data of the bundle is expected to be found in a body of a message.

By default bundles are installed into `/opt/eclipse/kura/deploy` directory, which is scanned by the Felix File Install
support installed by the
[Rhiot kura-install-felix-fileinstall command](../../tooling/cmd.md#kurainstallfelixfileinstall).

So for the example route above, sending bundle jar file to the `deploymanager/install/mybundle-1.0` topic will end up
in installing `/opt/eclipse/kura/deploy/mybundle-1.0.jar` into the device file system.

## Uninstalling bundles

In order to uninstall bundles from the Felix Install directory (defaulr `/opt/eclipse/kura/deploy`) via MQTT topic, deploy
the following route into your gateway device:

    import io.rhiot.component.kura.deploymanager.UninstallProcessor;

    ...

    from("paho:deploymanager/uninstall").
        process(new UninstallProcessor());

The consumer part of the route can be any Camel component. The important is that a body of a message sent to the uninstall
processor should be convertable to String indicating the name of bundle to uninstall.

So for the example route above, sending `guava-18.0` String to the `deploymanager/uninstall` topic will end up
in uninstalling `/opt/eclipse/kura/deploy/guava-18.0.jar` bundle from a device file system.