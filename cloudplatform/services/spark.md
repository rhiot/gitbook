# Apache Spark service

Apache Spark service allows you to bridge a long-running clustered [Apache Spark](http://spark.apache.org) application with an
IoT Connector. Spark service embedded in a driver application listens to requests coming to the IoT Connector and binds those with submitted RDD definition
and job definition. A payload sent to the Apache Spark service is passed as an input to the requested job.

<img src="rhiot_cloud_platform_service_spark.png" align="center" height="300">

Under the hood, Spark Service uses [Apache Camel Spark component](http://camel.apache.org/apache-spark) to bind jobs
invocations requests with RDD definitions.

The in order to execute the defined job named `rddCallback` against defined RDD `rdd` with given input data `input`,
send a message to the following IoT Connector Channel:

    input -> spark.rdd.rddCallback

A message returned back as a response from a channel contains a result of a Spark job computations.

## Running Spark service in a Spring Boot runtime environment

In order to create Spark service application, add these jars to your Spark application Maven project:

 			<dependency>
 				<groupId>io.rhiot</groupId>
 				<artifactId>rhiot-cloudplatform-runtime-spring</artifactId>
 				<version>${rhiot.version}</version>
 			</dependency>
            <!-- Choose some payload encoding jar -->
 			<dependency>
 				<groupId>io.rhiot</groupId>
 				<artifactId>rhiot-cloudplatform-encdoing-json</artifactId>
 				<version>${rhiot.version}</version>
 			</dependency>
			<dependency>
				<groupId>io.rhiot</groupId>
				<artifactId>rhiot-cloudplatform-service-spark</artifactId>
				<version>${rhiot.version}</version>
			</dependency>

Spring Boot runtime automatically detects and starts Spark service as soon `CloudPlatform` instance is started:

    import io.rhiot.cloudplatform.runtime.spring.CloudPlatform;
    ...
    new CloudPlatform().start();

By default Spark service attempts to connect to IoT Connector by resolving `AMQP_SERVICE_HOST` and `AMQP_SERVICE_PORT`
properties with defaults (`localhost:5672`). It means that in Kubernetes environment your application just connects to the AMQP broker.
For non-Kubernetes environment you usually have to configure those properties.

In order to change default master URL (`local[*]`), set the `spark.masterUrl` property.

## Registering jobs in a Spring Boot runtime environment

In order to register an RDD in a Spring Boot application register it as a bean:

    @Bean
    JavaRDD myRDD(JavaSparkContext sparkContext) {
        return sparkContext.textFile("/data/testrdd.txt");
    }

To register a job logic, create a Spring bean implementing Camel
[RddCallback](https://github.com/apache/camel/blob/master/components/camel-spark/src/main/java/org/apache/camel/component/spark/RddCallback.java)
interface:

    import static org.apache.camel.component.spark.annotations.AnnotatedRddCallback.annotatedRddCallback;

    ...

    @Bean
    Object myCallback() {
        return annotatedRddCallback(new Object(){
            @RddCallback
            public long onRdd(AbstractJavaRDDLike rdd, int multiplier) {
                return multiplier * rdd.count();
            };
        });
    }

Refer to the [Apache Camel Spark component documentation](http://camel.apache.org/apache-spark) to learn more about
Camel Spark API.

The in order to execute the defined job (`myCallback`) against defined RDD (`myRDD`) with given input data (`multiplier`)
send a message to the following IoT Connector Channel:

    int multiplier -> spark.myRdd.myCallback

The message returned back as a response to the channel will contain the result of the Spark job computations.

## Configuration of Spark service in a Spring Boot runtime environment

In order to change default driver application name (`Spring Boot Spark Application`), set the value of `spark.applicationName`
property. To set Spark home on your Spark context, set the `spark.sparkHome` property.