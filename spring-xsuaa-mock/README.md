# XSUAA Security Xsuaa Mock Library

## Description
This library enhances the spring-xsuaa project. This includes a `XsuaaMockWebServer` web server for the Xsuaa service that can provide *token_keys* for an offline JWT token validation. This is required only when there is no Xsuaa service (OAuth resource-server) in place, which is only the case in context of unit tests, as well as when running your Spring boot application locally.

The default implementation offers already valid *token_keys* for JWT tokens, that are generated by the [`JwtGenerator`](/spring-xsuaa-test/src/main/java/com/sap/cloud/security/xsuaa/test/JwtGenerator.java) (`spring-xsuaa-test` library).

## Requirements
- Java 8
- maven 3.3.9 or later
- Spring Boot 2.1 and later

## Configuration

### Maven Dependency
```xml
<dependency>
    <groupId>com.sap.cloud.security.xsuaa</groupId>
    <artifactId>spring-xsuaa-mock</artifactId>
    <version>1.2.0</version>
</dependency>
```

### Setup Mock Web Server
Add the following class, which makes sure, that the Xsuaa mock web server is only started in case a dedicated profile e.g. `uaamock` is active. Make sure that this profile (`uaamock`) is never active in production!

```java
public class XsuaaMockPostProcessor implements EnvironmentPostProcessor, DisposableBean {

    private final XsuaaMockWebServer mockAuthorizationServer = new XsuaaMockWebServer();

    @Override
    public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
        if (environment.acceptsProfiles(Profiles.of("uaamock"))) {
            environment.getPropertySources().addFirst(this.mockAuthorizationServer);
        }
    }

    @Override
    public void destroy() throws Exception {
        this.mockAuthorizationServer.destroy();
    }
}
```

Then you have to register this class to `META-INF/spring.factories`:

```
org.springframework.boot.env.EnvironmentPostProcessor=<<your package>>.XsuaaMockPostProcessor
```


And finally you need to configure the `spring.security.oauth2.resourceserver.jwt.jwk-set-uri`, the `xsuaa.url` and the `xsuaa.identityzoneid` properties in the same way as done in this [`application-uaamock.properties`](src/test/resources/application-uaamock.properties) file.


### Extendability
Note: it is possible to extend the dispatcher and pass this to the `XsuaaMockWebServer` constructor. An example `XsuaaMockPostProcessor` implementation can be found [here](src/test/java/com/sap/cloud/security/xsuaa/mock/XsuaaMockPostProcessor.java).
