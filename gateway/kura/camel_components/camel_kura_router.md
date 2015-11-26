# Camel Kura router

**This component can be used only in the Kura server.**

**Avaliable since Rhiot 0.1.3**: Rhiot provides `io.rhiot.component.kura.router.RhiotKuraRouter` class, which extends 
`org.apache.camel.component.kura.KuraRouter` class from the Apache Camel 
[camel-kura](http://camel.apache.org/kura) module. While `KuraRouter` provides a generic base for Kura routes, it 
doesn't rely on the Kura-specific jars, because of limitations of the Apache Camel policy regarding adding 3rd parties repositories to the Camel (like Eclipse Kura repository). `RhiotKuraRouter` extends `KuraRouter` and enhances it with Kura-specific API.

## Maven dependency

Maven users should add the following dependency to their POM file:

    <dependency>
      <groupId>io.rhiot</groupId>
      <artifactId>camel-kura</artifactId>
      <version>${rhiot.version}</version>
    </dependency>

Adding Rhiot camel-kura module to your project, imports transitive Kura dependencies. This is big advantage over Apache
Camel camel-kura module, which doesn't rely on Kura API and therefore doesn't import Kura jars.

## Usage

The principle of using `RhiotKuraRouter` is the same as using `KuraRouter` i.e. just extend `RhiotKuraRouter` class:

    import io.rhiot.component.kura.router.RhiotKuraRouter;
    
    class TestKuraRouter extends RhiotKuraRouter {

      @Override
	  public void configure() throws Exception {
	    from("direct:test")
	     .to("mock:test");
	  }

	}

## Loading XML routes using SCR property

`RhiotKuraRouter` comes with a `RhiotKuraRouter#updated(Map<String, Object>)` method. The primary purpose of this callback 
is to allow a router to be a SCR component configured using the EuroTech 
[Everyware Cloud](http://www.eurotech.com/en/products/software+services/everyware+cloud+m2m+platform/m2m+what+it+is),
however you can use this callback outside the Everyware Cloud as well.

Whenever `RhiotKuraRouter#updated(Map<String, Object>)` callback is executed, `RhiotKuraRouter` tries to read `camel.route.xml`
property value (`RhiotKuraRouter.XML_ROUTE_PROPERTY` key constant), to parse its value and load it as an XML Camel routes. 
For example if the `camel.route.xml` property will be set to the following value...

    <routes xmlns="http://camel.apache.org/schema/spring">
        <route id="mqttLogger">
            <from uri="paho:topic?brokerUrl=tcp:brokerhost:1883"/>
            <to uri="log:messages"/>
        </route>
    </routes>
    
...new route will be automatically started (or updated if route with ID equal to `mqttLogger` already exists).
	
## Camel Kura PojoSR test facility

This facility is intended to be used to test Camel routes to be deployed on a Kura server. Just instantiate a new `io.rhiot.component.kura.PojosrKuraServer` and call its `start` method passing your Camel route.

	PojosrKuraServer pojosrKuraServer = new PojosrKuraServer();
    MyKuraRouter startedKuraRouter = kura.start(MyKuraRouter.class);

This will instantiate an OSGi registry, add your Camel route and its dependencies to the classpath and execute your route.