# Camel Kura router

**Avaliable since Rhiot 0.1.3**: Rhiot provides `io.rhiot.component.kura.router.RhiotKuraRouter` class, which
extends `org.apache.camel.component.kura.KuraRouter` class from the Apache Camel 
[camel-kura](http://camel.apache.org/kura) module. While `KuraRouter` provides a generic base for Kura routes, it doesn't
rely on the Kura-specific jars, because of limitations of the Apache Camel policy regarding adding 3rd parties repositories
to the Camel (like Eclipse Kura repository). `RhiotKuraRouter` extends `KuraRouter` and enhances it with Kura-specific API.

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
