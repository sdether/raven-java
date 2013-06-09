# Raven (module)
Main module of the Raven project in java. It provides a client to send messages
to a Sentry server as well as an implementation of an [`Handler`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/Handler.html)
for `java.util.logging`.

## Installation

### Maven
```xml
<dependency>
    <groupId>net.kencochrane.raven</groupId>
    <artifactId>raven</artifactId>
    <version>4.0</version>
</dependency>
```

### Other dependency managers
Details in the [central Maven repository](http://search.maven.org/#artifactdetails%7Cnet.kencochrane.raven%7Craven%7C4.0%7Cjar).

### Manual dependency management
Relies on:

 - [raven-4.0.jar](http://search.maven.org/#artifactdetails%7Cnet.kencochrane.raven%7Craven%7C4.0%7Cjar)
 - [slf4j-api-1.7.5.jar](http://search.maven.org/#artifactdetails%7Corg.slf4j%7Cslf4j-api%7C1.7.5%7Cjar)
 - [commons-codec-1.8.jar](http://search.maven.org/#artifactdetails%7Ccommons-codec%7Ccommons-codec%7C1.8%7Cjar)
 - [jackson-core-2.2.2.jar](http://search.maven.org/#artifactdetails%7Ccom.fasterxml.jackson.core%7Cjackson-core%7C2.2.2%7Cjar)


## Usage (`java.util.logging`)
### Configuration
In the `logging.properties` file set:

```properties
level=INFO
handlers=net.kencochrane.raven.jul.SentryHandler
net.kencochrane.raven.jul.SentryHandler.dsn=http://publicKey:secretKey@host:port/1?options
```

When starting your application, add the `java.util.logging.config.file` to the
system properties, with the full path to the `logging.properties` as its value.

    $ java -Djava.util.logging.config.file=/path/to/app.properties MainClass

### In practice
```java
import java.util.logging.Level;
import java.util.logging.Logger;

public class MyClass {
    private static final Logger logger = Logger.getLogger(MyClass.class.getName());

    void logSimpleMessage() {
        // This adds a simple message to the logs
        logger.log(Level.INFO, "This is a test");
    }

    void logException() {
        try {
            unsafeMethod();
        } catch (Exception e) {
            // This adds an exception to the logs
            logger.log(Level.SEVERE, "Exception caught", e);
        }
    }

    void unsafeMethod() {
        throw new UnsupportedOperationException("You shouldn't call that");
    }
}
```

### Unsupported features
`java.util.logging` does not support either MDC nor NDC, meaning that it is not
possible to attach additional/custom context values to the logs.
In other terms, it is not possible to use the "extra" field supported by Sentry.


## Manual usage (NOT RECOMMENDED)
It is possible to use the client manually rather than using a logging framework
in order to send messages to Sentry. It is not recommended to use this solution
as the API is more verbose and requires the developer to specify the value of
each field sent to Sentry.

### In practice
```java
import net.kencochrane.raven.Raven;
import net.kencochrane.raven.RavenFactory;
import net.kencochrane.raven.event.Event;
import net.kencochrane.raven.event.EventBuilder;
import net.kencochrane.raven.event.interfaces.ExceptionInterface;
import net.kencochrane.raven.event.interfaces.MessageInterface;

public class MyClass {
    private static Raven raven;

    public static void main(String... args) {
        // Creation of the client with a specific DSN
        String dsn = args[0];
        raven = RavenFactory.ravenInstance(dsn);

        // It is also possible to use the DSN detection system like this
        raven = RavenFactory.ravenInstance();
    }

    void logSimpleMessage() {
        // This adds a simple message to the logs
        EventBuilder eventBuilder = new EventBuilder()
                        .setMessage("This is a test")
                        .setLevel(Event.Level.INFO)
                        .setLogger(MyClass.class.getName());
        raven.runBuilderHelpers(eventBuilder); // Optional
        raven.sendEvent(eventBuilder.build());
    }

    void logException() {
        try {
            unsafeMethod();
        } catch (Exception e) {
            // This adds an exception to the logs
            EventBuilder eventBuilder = new EventBuilder()
                            .setMessage("Exception caught")
                            .setLevel(Event.Level.ERROR)
                            .setLogger(MyClass.class.getName())
                            .addSentryInterface(new ExceptionInterface(e));
            raven.runBuilderHelpers(eventBuilder); // Optional
            raven.sendEvent(eventBuilder.build());
        }
    }

    void unsafeMethod() {
        throw new UnsupportedOperationException("You shouldn't call that");
    }
}
```