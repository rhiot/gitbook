# Camel OpenIMAJ component

Camel OpenIMAJ component can be used to detect faces in images.
Camel OpenIMAJ component supports only producer endpoints.

## Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-openimag</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

 Avaliable for rhiot.version >= 0.1.3

## URI format

    openimaj:label

Where `label` can be replaced with any text label:

    from("webcam:spycam").to("openimaj:face-detection");

This routes the input stream from the webcam to the openimaj component,
when a face is detected the resulting body will be an instance of org.openimaj.image.processing.face.detection.DetectedFace,
and List<DetectedFace> when there are multiple faces.

The component uses a face detector based on the Haar cascade by default, optionally set an alternate detector;

    from("webcam:spycam").to("openimaj:face-detection?faceDetector=#anotherDetector");

Using the ProducerTemplate, the following example uses the FKEFaceDetector which is a wrapper around the Haar detector, providing additional
information by finding facial keypoints on top of the face;

    KEDetectedFace face = template.requestBody("openimaj:face-detection?faceDetector=#fkeFaceDetector", inputStream, KEDetectedFace.class);
    FacialKeypoint keypoint = face.getKeypoint(FacialKeypoint.FacialKeypointType.EYE_LEFT_LEFT);


The confidence in the face detection is set to the low default of 1, you can set the minimum confidence threshold on the endpoint;

    from("webcam:spycam").to("openimaj:face-detection?confidence=50");


### URI Options Parameters

| Option                    | Default value                                                                 | Description   |
|:------------------------- |:-----------------------------------------------------------------------       |:------------- |
| `faceDetector`            | `new HaarCascadeDetector()`                                                   | `org.openimaj.image.processing.face.detection.HaarCascadeDetector' is the default face detector |
| `confidence`              | 1                                                                             | The minimum confidence of the detection; higher numbers mean higher confidence.      |


