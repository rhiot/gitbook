# Camel OpenALPR component

Camel OpenALPR component relies on the [OpenALPR](openalpr.com) library to perform image recognition of the incoming
binary data in order to find and read license plates.

## Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>rhiot-cloudplatform-camel-openalpr</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

**Available for `rhiot.version` >= 0.1.4**

## URI format

    openalpr:label

Where `label` can be replaced with any text label:

    from("webcam:plateCam").to("openalpr:plateScanner");

This routes the input stream from the webcam to the OpenALPR component. The results of the scanning are represented
as a list of `io.rhiot.cloudplatform.camel.openalpr.PlateMatch`:

    import io.rhiot.cloudplatform.camel.openalpr.PlateMatch;
    ...
    InputStream image = ...;
    List<PlateMatch> matches = producerTemplate.requestBody("openalpr:plateScanner", image, List.class);
    PlateMatch bestMatch = matches.get(0);
    String plateNumer = bestMatch.getPlateNumber();
    double matchConfidenceInPercent = bestMatch.getConfidence();

## Dockerized OpenALPR

By default this component relies on a dockerized version of the OpenALPR. It means that you should have Docker
shell client installed and configured.

You can change the system command used by the component to invoke OpenALPR, by setting the `commandStrategy` option on
the endpoint and referencing custom `io.rhiot.cloudplatform.camel.openalpr.OpenalprCommandStrategy` instance:

    @Bean
    OpenalprCommandStrategy myCommand() {
        return new OpenalprCommandStrategy(){
            String[] openalprCommand(OpenalprEndpoint openalprEndpoint, File imageFile) {
                return new String[]{"alpr", "-c", "eu", imageFile.getName()};
            }
        };
    }

    ...


    from(...).to("openalpr:scan?commandStrategy=#myCommand");

## URI Options Parameters

| Option                    | Default value                                                                 | Description   |
|:------------------------- |:-----------------------------------------------------------------------       |:------------- |
| `country`            | `eu`                                                   | `Country class of the plates. Can be `eu` or `us`. |
| `workDir`              | `openalpr-workdir`      | Working directory used by OpenALPR to temporarily store files for the analysis.      |
| `processManager`  | `new io.rhiot.utils.process.DefaultProcessManager` | `io.rhiot.utils.process.ProcessManager` instance used to execute OpenALPR command. |
| `commandStrategy` |   `new io.rhiot.cloudplatform.camel.openalpr.DockerizedOpenalprCommandStrategy`   | `io.rhiot.cloudplatform.camel.openalpr.OpenalprCommandStrategy` instance used to generate OpenALPR system command. |

