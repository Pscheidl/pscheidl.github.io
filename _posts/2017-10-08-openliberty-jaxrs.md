---
title: "OpenLiberty.io: Simple guide"
published: true
categories:
  - JavaEE
  - EE4J
tags:
  - JavaEE
  - CDI
  - JAX-RS
  - OpenLiberty
---

*Sample application is built as a part of this article. It can be found on [GitHub](https://github.com/Pscheidl/openliberty-jaxrs-example)*.

Year 2017 and we're all building microservices, where shared runtime environment represented by a single application server hosting many applications is [passé](https://www.merriam-webster.com/dictionary/pass%C3%A9). As a result, rightsizing application environment makes sense ! The functionality is still required, but not all at once by a single application, if ever. Therefore, the packaging and distribution changed, as mentioned in one of my [previous posts](http://www.pavel.cool/javaee/java-ee-fatjars-docker/). However, the obvious and most simple way to just pack everything into one Fatjar comes with long redeployment times and developers stuck waiting for something to reload, when an application server provides the environment already running and redeploying the business logic takes just few millisecons. A Java rockstar Adam Bien von München is pointing this fact out for a long and if you'd like to know more, watch his video on [Thin WARs, Java EE 7, Docker and Productivity](https://www.youtube.com/watch?v=5N4EUDhrkec). Java EE offers many solutions so far.

- [WildFly Swarm](http://wildfly-swarm.io/)
- [Payara Micro](https://www.payara.fish/downloads)
- [KumuluzEE](https://ee.kumuluz.com/)
- [Meecrowave](http://openwebbeans.apache.org/meecrowave/)

And now, one more player appears. It's called [OpenLiberty](https://openliberty.io/) by IBM. OpenLiberty.io is completely open source and by their own words, it is used to build cloud-native apps and microservices while running only what you need. At the moment, it provides full Java EE 7 stack complemented with the mighty [MicroProfile](https://github.com/eclipse/microprofile). Java EE 8 support is [being added](https://developer.ibm.com/wasdev/blog/2017/09/29/microprofile-metrics-sept-2017-beta/) constantly. New JAX-RS or Servlet 4.0 are already supported.

OpenLiberty.io [addresses](https://openliberty.io/about/) a common fear of many developers - long reload times. It is fast to start and provides dynamic code update capabilities.

## Building sample application

I belive I've [written enough](http://www.pavel.cool/javaee/java-ee-fatjars-docker/) theory articles on packaging, fast redeployments and good engineering practices in Java application packaging. Time to offer something practical. Commercial break: If you want to read more on both theory and practice, our new book [Java EE 8 MicroServices](https://www.packtpub.com/application-development/java-ee-8-microservices) is available for pre-order. Spring Boot is covered in the book.

It is **absolutely easy** to create a Java EE microservice running on OpenLiberty.  The application described is ready to `git clone` and run on [GitHub](https://github.com/Pscheidl/openliberty-jaxrs-example).

1. Create a common Java EE application.
1. Add OpenLiberty configuration to Maven's pom.xml
1. Configure server properties with OpenLiberty's server.xml

Two additional steps are required to be taken. OpenLiberty, as any other solution on the market, needs to attach to the microservice build process in order to produce the rightsized artifact. In order for OpenLiberty to download all the necessary parts, attach on proper ports and instantiate the right services, a simple configuration has to be added. There is nothing more.

### Creating Maven project

First a Java EE Maven project has to be created.


```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cool.pavel</groupId>
    <artifactId>openliberty-jaxrs-example</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <endorsed.dir>${project.build.directory}/endorsed</endorsed.dir>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>


    <dependencies>
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>7.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.2</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <compilerArguments>
                        <endorseddirs>${endorsed.dir}</endorseddirs>
                    </compilerArguments>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.1.0</version>
                <configuration>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                    <packagingExcludes>pom.xml</packagingExcludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
````

There is only one provided dependency referencing Java EE 7 functionality. Nothing more. The resulting application/microservice in a form of a Java Web ARchive (WAR) only contains business logic and is very thin so far. On top of this, OpenLiberty.io functionality is added. There are two tings to enable OpenLiberty in a Maven Java EE project.

1. OpenLiberty parent project
1. OpenLiberty plugin

Including OpenLiberty parent project in Maven is as simple as adding five lines into `pom.xml`.

```xml
    <parent>
        <groupId>net.wasdev.wlp.maven.parent</groupId>
        <artifactId>liberty-maven-app-parent</artifactId>
        <version>2.0</version>
    </parent>
```

As a last step, OpenLiberty's Maven plugin must be added. As seen from the following example, there are several values like OpenLiberty runtime version or port configurations. Following good manners, these variables are externalized into Maven's properties section.

```xml
            <plugin>
                <groupId>net.wasdev.wlp.maven.plugins</groupId>
                <artifactId>liberty-maven-plugin</artifactId>
                <version>2.0</version>
                <configuration>
                    <assemblyArtifact>
                        <groupId>io.openliberty</groupId>
                        <artifactId>openliberty-runtime</artifactId>
                        <version>17.0.0.3</version>
                        <type>zip</type>
                    </assemblyArtifact>
                    <serverName>${project.artifactId}Server</serverName>
                    <stripVersion>true</stripVersion>
                    <configFile>src/main/liberty/config/server.xml</configFile>
                    <packageFile>${package.file}</packageFile>
                    <include>${packaging.type}</include>
                    <bootstrapProperties>
                        <default.http.port>${testServerHttpPort}</default.http.port>
                        <default.https.port>${testServerHttpsPort}</default.https.port>
                        <app.context.root>${project.artifactId}</app.context.root>
                    </bootstrapProperties>
                </configuration>
                <executions>
                    <execution>
                        <id>package-server</id>
                        <phase>package</phase>
                        <goals>
                            <goal>package-server</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>target/wlp-package</outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```

The resulting `<properties>...</properties>` section with externalized variables ends up as follows. Runtime version, OpenLiberty server ports and package path are specified in addition to common Maven properties.

```xml
    <properties>
        <endorsed.dir>${project.build.directory}/endorsed</endorsed.dir>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <testServerHttpPort>9080</testServerHttpPort>
        <testServerHttpsPort>9443</testServerHttpsPort>
        <package.file>${project.build.directory}/${project.artifactId}.zip</package.file>
        <packaging.type>usr</packaging.type>
        <openliberty.runtime.version>17.0.0.3</openliberty.runtime.version>
    </properties>
```

#### Final Pom.xml

The final and complete pom.xml with OpenLiberty setup and Java EE 7 functionality referenced is listed below. When copy&pasting this file, do not be surprised the artifact groupId is `cool.pavel` and artifactId is `liberty-maven-app-parent`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>net.wasdev.wlp.maven.parent</groupId>
        <artifactId>liberty-maven-app-parent</artifactId>
        <version>2.0</version>
    </parent>

    <groupId>cool.pavel</groupId>
    <artifactId>openliberty-jaxrs-example</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <endorsed.dir>${project.build.directory}/endorsed</endorsed.dir>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <testServerHttpPort>9080</testServerHttpPort>
        <testServerHttpsPort>9443</testServerHttpsPort>
        <package.file>${project.build.directory}/${project.artifactId}.zip</package.file>
        <packaging.type>usr</packaging.type>
        <openliberty.runtime.version>17.0.0.3</openliberty.runtime.version>
    </properties>


    <dependencies>
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>7.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.2</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <compilerArguments>
                        <endorseddirs>${endorsed.dir}</endorseddirs>
                    </compilerArguments>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.1.0</version>
                <configuration>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                    <packagingExcludes>pom.xml</packagingExcludes>
                </configuration>
            </plugin>
            <plugin>
                <groupId>net.wasdev.wlp.maven.plugins</groupId>
                <artifactId>liberty-maven-plugin</artifactId>
                <version>2.0</version>
                <configuration>
                    <assemblyArtifact>
                        <groupId>io.openliberty</groupId>
                        <artifactId>openliberty-runtime</artifactId>
                        <version>${openliberty.runtime.version}</version>
                        <type>zip</type>
                    </assemblyArtifact>
                    <serverName>${project.artifactId}Server</serverName>
                    <stripVersion>true</stripVersion>
                    <configFile>src/main/liberty/config/server.xml</configFile>
                    <packageFile>${package.file}</packageFile>
                    <include>${packaging.type}</include>
                    <bootstrapProperties>
                        <default.http.port>${testServerHttpPort}</default.http.port>
                        <default.https.port>${testServerHttpsPort}</default.https.port>
                        <app.context.root>${project.artifactId}</app.context.root>
                    </bootstrapProperties>
                </configuration>
                <executions>
                    <execution>
                        <id>package-server</id>
                        <phase>package</phase>
                        <goals>
                            <goal>package-server</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>target/wlp-package</outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>


</project>
```

### Sample JAX-RS endpoint

In order to test OpenLiberty is up&running, a simple RESTful resource using JAX-RS is just right. Create a class `Resea`

```java
package cool.pavel.openliberty.api;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.core.Response;

@Path("/ping")
public class HealthCheck {

    @GET
    public Response ping() {
        return Response.ok("Pong")
                .build();
    }
}

```

Since JAX-RS is used, the RESTful API path is configured by creating a class extending `javax.ws.rs.core.Application`.

```java
package cool.pavel.openliberty.api;

import javax.ws.rs.ApplicationPath;
import javax.ws.rs.core.Application;

@ApplicationPath("")
public class MicroserviceConfiguration extends Application {

}

```


Previous two classes are placed in `cool.pavel.openliberty.api` package.

```text
main
│   ├── java
│   │   └── cool
│   │       └── pavel
│   │           └── openliberty
│   │               └── api
│   │                   ├── HealthCheck.java
│   │                   └── MicroserviceConfiguration.java
```


### Configuring OpenLiberty server
In order to configure OpenLiberty server, create `server.xml` file in `{project-root}/src/main/liberty/config`.


```xml
<server description="Sample Liberty server">

  <featureManager>
      <feature>jaxrs-2.0</feature>
  </featureManager>

  <httpEndpoint httpPort="${default.http.port}" httpsPort="${default.https.port}"
      id="defaultHttpEndpoint" host="*" />

  <webApplication location="openliberty-jaxrs-example-1.0-SNAPSHOT.jar" contextRoot="${app.context.root}"/>
</server>
```


The `<featureManager>` part contains a list of features to be included in the created runtime. Basic configuration also contains default host and port bindings and path to applications to be contained in the resulting package. Besides `<webApplication ...>` tag, it is also possible to use `<enterpriseApplication ...>` tag to deploy Enterprise Archives, if required.

With the `server.xml` file, the `main` folder's content looks as follows. Nothing more is added to the project in order to run it. In this state, the application/microservice is fully functional.


```text
.
├── main
│   ├── java
│   │   └── cool
│   │       └── pavel
│   │           └── openliberty
│   │               └── api
│   │                   ├── HealthCheck.java
│   │                   └── MicroserviceConfiguration.java
│   ├── liberty
│   │   └── config
│   │       └── server.xml

```



## Invoking & fast redeployment

In order to run the application, execute `mvn package liberty:run-server` command. OpenLiberty is attached to Maven's package goal and automatically downloads the dependencies necessary to package the final application. The `liberty:run-server` goal starts OpenLiberty server with configuration defined in `server.xml` and deploys all applications defined in the very same `server.xml` file.

As OpenLiberty starts, important information appear on `stdout`.

```text
...
[INFO] [AUDIT   ] CWWKT0016I: Web application available (default_host): http://localhost:9080/openliberty-jaxrs-example/
[INFO] [AUDIT   ] CWWKZ0001I: Application openliberty-jaxrs-example started in 0,222 seconds.
...
```

First, OpenLiberty tells the URL under which the application is reachable. In case of the sample application, it is `http://localhost:9080/openliberty-jaxrs-example/`. Secondly, it tells the time required for an application to start. In this case 0,222 seconds. 200 milliseconds for a complete application startup is great result.

When invoking HTTP GET on `http://localhost:9080/openliberty-jaxrs-example/ping`, a HTTP 200 OK response arrives with "Pong" message in body.

### Redeploy

Now comes the fun part. Let's change the endpoint and see how fast OpenLiberty updates the application with the latest changes. A new method producing response message is introduced to make distinguish OpenLiberty from simple HotSwap solutions. Also, the response changes from HTTP 200 OK to HTTP 202 Accepted. The final shape of the `HealthCheck` class can be found in the figure below.


```java
package cool.pavel.openliberty.api;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.core.Response;

@Path("/ping")
public class HealthCheck {

    @GET
    public Response ping() {
        return Response.accepted(getMessage())
                .build();
    }

    private String getMessage(){
        return "A request for ping has been accepted.";
    }
}

```

Now, simply **recompile** the application. This can be easily done by invoking `mvn compile`. OpenLiberty takes care about everything else. The result can be seen in OpenLiberty's console.


```text
[INFO] [WARNING ] CWWKZ0014W: The application openliberty-jaxrs-example-1.0-SNAPSHOT could not be started as it could not be found at location openliberty-jaxrs-example-1.0-SNAPSHOT.jar.
[INFO] [AUDIT   ] CWWKT0016I: Web application available (default_host): http://10.0.0.13:9080/openliberty-jaxrs-example/
[INFO] [AUDIT   ] CWWKZ0001I: Application openliberty-jaxrs-example started in 0,222 seconds.
[INFO] [AUDIT   ] CWWKF0012I: The server installed the following features: [servlet-3.1, json-1.0, jaxrs-2.0, jaxrsClient-2.0].
[INFO] [AUDIT   ] CWWKF0011I: The server openliberty-jaxrs-exampleServer is ready to run a smarter planet.
[INFO] [AUDIT   ] CWWKT0017I: Web application removed (default_host): http://10.0.0.13:9080/openliberty-jaxrs-example/
[INFO] [AUDIT   ] CWWKZ0009I: The application openliberty-jaxrs-example has stopped successfully.
[INFO] [AUDIT   ] CWWKT0016I: Web application available (default_host): http://10.0.0.13:9080/openliberty-jaxrs-example/
[INFO] [AUDIT   ] CWWKZ0003I: The application openliberty-jaxrs-example updated in 0,035 seconds.
```

It took only **35 milliseconds** to redeploy the changes. After invoking HTTP GET on `http://localhost:9080/openliberty-jaxrs-example/ping`, a HTTP 202 Accepted response arrives with "A request for ping has been accepted" message in body. No need to redeploy manually, no need for external functionality or incorporating third-party tools.

The results may vary from machine to machine, however there is no technology able to reach comparable results on my machine so far. Fast redeployments are varying from 200 ms to one second. Spring Loaded is even slower and [technologically differnt/limited](https://martinsdeveloperworld.wordpress.com/2015/05/02/updating-code-at-runtime-spring-loaded-demystified/).

## Comparison & final thoughts

OpenLiberty.io is pretty impressive. It has everything. Whole Java EE 7 stack with growing support for Java EE 8 and most importantly, Eclipse MicroProfile. Startup and automatic updates are a fresh breeze. Especially Spring Boot developers should try it. And no, Spring Loaded does not even remotely match capabilities and speed of OpenLiberty. OpenLiberty also makes the usual advantage of fast redeployments to traditional application servers disappear.

However, There were few things I was missing or disliked. But please note OpenLiberty.io is a very young project and things will probably get better any day. First, there is no roadmap to Java EE 8 to be found. I also disliked the XML configuration. Different and more human-friendly formats are available. It is however not a deal breaker, especially when Maven is used. Also, a reference manual like WildFly Swarm, Spring Boot or Payara Micro has (Payara has an excellent one) on OpenLiberty.io website would be nice.

Of course, there are other solutions out there on the market. Each one of them has slightly different approach and the diversity gives everyone the opportunity to find the best fitting solution. However, I must admit, OpenLiberty, eventhough being a fresh project, is a serious player and a joy to work with.

### WildFly Swarm

Pros
- Automatic scanning, no need for server.xml
- Reference guide
- Project generator
- Better tutorials
- Monthly releases
- Hollow jars

Cons
- Building fatjar is significantly slower process
- Redeployment is significantly slower process

### Payara Micro

Pros
- No additional configuration by default
- Outstanding reference guide, better tutorials
- No project generator required
- Official support available
- Ability to `java -jar payara-micro.jar --deploy application.war`

Cons
- Bigger fatjar

### Spring Boot

Pros
- Cleaner pom.xml in recent versions
- Excellent Gradle support
- Spring ?

Cons
- Spring ? Mainly the configuration bloatware (@ComponentScan, @EnableAutoConfiguration ...)
- Slightly bigger in size
- "Fast reloads complicated are, JRebel required may be" - Mr. Yoda on Spring Boot
- No MicroProfile-like abstraction, cloud-related functionality depends on specific solutions
