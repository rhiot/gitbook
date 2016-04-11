# PaaS

Rhiot PaaS is an opinionated, Docker container based, installation of the Rhiot cloud. All you have to to in order
to start Cloud PaaS is [executing the installation script](../starting.md). As soon as startup script is executed, 
you can take an advantage of preconfigured installation of Cloud PaaS.

## Loading custom startup script

PaaS script looks up for the `~/.rhiot/paas.config` script file when starting PaaS platform. If the file exists, it will
be interpreted as a Bash script and executed. So for example to start a csutom Docker container when starting PaaS, add
the following line to the `~/.rhiot/paas.config` file:

    docker run -d example/myCustomImage:1.0

## Security

PaaS installation comes with an instance of a [KeyCloak](http://keycloak.jboss.org) server. The KeyCloak endpoint is
used to manage users authorized to connect to cloud event bus.

The KeyCloak web application is exposed on `CLOUDHOST:8082/auth/admin/REALM/console` address. In particular
`CLOUDHOST:8082/auth/admin/master/console/` address can be used to manage master realm.

## Apache Spark cluster

We treat Apache Spark as a first class citizen of Cloud Platform. This is why when you start the PaaS script, the latter
starts Spark master and worker instances for you. Spark web UI is available under following URL - `http://localhost:8081`.

To submit a Spark task to the PaaS cluster, execute the following command on your server:

    bash <(curl -sL https://goo.gl/sQxjnE) --class com.example.MyJob  /tmp/jobs/myjob.jar

Keep in mind that a jar with your job must be located in the `/tmp/jobs` directory.

### Disabling Spark cluster

In order to disable Spark cluster when starting the PaaS, set `SPARK_ENABLED` environment variable to `false` before
executing the PaaS script:

    export SPARK_ENABLED=false

