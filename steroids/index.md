# Steroids

## Steroids bootstrap

Steroids Bootstrap is a small engine that can be used to scan the classpath and automatically load steroids modules.
Bootstrap provides opinionated *convention over configuration* runtime simplifying the wiring between common components used in Rhiot-based applications.

In order to start using bootstrap, add the following Maven dependency to your project:

    <dependency>
        <groupId>io.rhiot</groupId>
        <artifactId>rhiot-steroids</artifactId>
        <version>${rhiot.version}</version>
    </dependency>

Avaliable for rhiot.version >= 0.1.2


And then add the following code to your project:

    import io.rhiot.steroids.bootstrap.Bootstrap;
    ...
    Bootstrap bootstrap = new Bootstrap().start();
    ... // Do your stuff
    bootstrap.stop();

You can also use our main class (which is particularly useful when working with the fat jars):

    import io.rhiot.steroids.bootstrap.Bootstrap;
    ...
    Bootstrap.main();

### Injecting MongoDB client

Steroids come with the MongoDB module that can be used to simplify access to the MongoDB database. In order to take the
advantage from it, import the `rhiot-mongodb` module into your project:

    <dependency>
        <groupId>io.rhiot</groupId>
        <artifactId>rhiot-mongodb</artifactId>
        <version>${rhiot.version}</version>
    </dependency>

Avaliable for rhiot.version >= 0.1.1

In order to inject the MongoDb client into your code, use the `Mongos.discoverMongo()` method:

    import io.rhiot.mongodb.Mongos;
    ...
    MongoClient mongo = Mongos.discoverMongo();

If `MONGODB_SERVICE_HOST` environment variable (or system property) is not specified, the `Mongos` will try to connect to
the `mongodb` and `localhost` hosts respectively, using default MongoDB port (`27017`) or the one specified by the `MONGODB_SERVICE_HOST`
environment variable (or system property).

By default MongoDB client will be configured to timeout the connection attempt after 1 second (yes, we like to fail fast).
You can change timeout value by setting `MONGODB_CONNECT_TIMEOUT` the environment variable (or system property) to the desired number
of timeout miliseconds. For example to set the connection timeout to 30 seconds you can use the following code:

    System.setProperty("MONGODB_CONNECT_TIMEOUT", TimeUnit.SECONDS.toMillis(30) + "");
    MongoClient mongo = Mongos.discoverMongo();
