# Monitoring of a mini-gateway

## Gateway logger configuration

**Available starting from Rhiot 0.1.4**

By default gateway keeps the last 100 MB of the logging history. Logs are grouped by the days and split into the
10 MB files. The default logging level is `INFO`. You can change it by setting the `log_level`configuration
property:

    log_level=DEBUG

By default logs are stored in the `/var/log` directory, where current log file is `/var/log/rhiot.log`. If you would like
to append logs to the standard output, instead of writing those to the file system, use the following configuration
setting:

    log_appender=STDOUT

## Monitoring gateway with Jolokia

The gateway exposes its JMX beans using the [Jolokia REST API](https://jolokia.org). The default Jolokia URL for the
gateway is `http://0.0.0.0:8778/jolokia`. You can take advantage of the Jolokia to monitor and perform administrative
tasks on your gateway.


