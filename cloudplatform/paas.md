# Cloud Platform PaaS

PaaS Cloud Platform is opinionated, Docker container based, installation of Cloud Platform. You just
[execute the installation script](starting.md) and can immediately take advantage if preconfigured installation of
Cloud Platform.

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

