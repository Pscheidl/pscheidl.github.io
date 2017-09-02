---
title: "Building, packaging and distributing Java EE applications in 2017"
published: true
categories:
  - JavaEE
tags:
  - JavaEE
  - Spring
  - Docker
  - Fatjar
---

A few decades ago, in this very galaxy, first computer languages and compilers were written [(compilers)](https://www.amazon.com/Compilers-Principles-Techniques-Tools-2nd/dp/0321486811), enabling developers to create programs faster and in comfort. Shortly, quite a lot of programs were created. The more observant programmers could not unsee that many problems repeat across the programs. The same solution to the very same problem was written again and again. The solution was simple -> write the repeating code once and only use references in the new code that was actually different and solved something new. For example GUI - many programs spawn new windows with forms inside them. Therefore, the code to create GUI elements was separated and reused.

However, that is not the end to our problems. As every developer knows, for program's instructions to be executed, they must be loaded into memory from a persistent. Then into CPU's memory and then executed by the CPU. So, developers were no longer writing repeating parts again and again, but those parts still had to be shipped with the code, because the resulting instructions were of course required for the program to run correctly. If the same example with GUI libraries is used, the library had to be become part of the resulting program translated by the compiler into something executable by the target machine. So, at compile time, actually in the last phases of compilation, those libraries were included in the resulting "executable program package". Such program could be distributed.

Of course, distributed, but with all the libraries. There was one computer, one operating systems, many programs and each and every one of them had all the libraries packed within. When those programs (art, in fact, parts of them) were loaded into memory to be executed, the same libraries were loaded over and over again. Back then and sometimes even now, memory is precious. In history, even hard drive space was precious. Thus, one more effort was made:

- Not to distribute all the libraries with the programs, but rather have them "downloaded" once in the whole system.
- Not to load all the same libraries many times into memory, but rather load the shared code once for other programs to use it.

These are, very very very briefly, basics of how programs and their distribution evolved. It has a lot to do with operating systems and the process of compiling the programs. If you would like to know more, there are several great books to start: [Operating Systems: Principles and Practice](https://www.amazon.com/Modern-Operating-Systems-Andrew-Tanenbaum/dp/013359162X/ref=sr_1_3?ie=UTF8&qid=1489860038&sr=8-3&keywords=operating+systems), [Compilers: Principles, techniques and tools](https://www.amazon.com/Compilers-Principles-Techniques-Tools-2nd/dp/0321486811).

Nowadays, for common business logic, there is plenty of memory and even more space on persistent storage. Also, the problem is not as shallow as I just presented it. If many libraries are pre-loaded in the system, those libraries may end up unused. Also, shared code has versions. Each program may require a slightly different version of "the same" library. Anyway, things can not be worse than **bundling everything everywhere**, but with the techniques mentioned above, chances are memory and disk space is saved. 

These are some very basic concepts even moderate people are fully aware of and I even guess some readers may be offended at this point. But please wait and read on, I myself have a point.

## Java EE and reusing code

In Java world, we have a variation on this problem. Every Java program requires a JVM (native code can still be created). So, there is one recent JVM on the machine and all the Java bytecode is ran by means of this one single JVM. The JVM is not distributed with each program. The story is the same with Java EE. Java EE represents a common effort of many vendors to standardize commonly used functionality and separate it from the actual business logic. Everyone is of course free to create the implementation, but that freedom solves different kinds of problems software industry faces time after time. In history, such common functionality, represented by separate JSRs, was bundled into a package called the "application server". There are many good intentions behind an application server:

- Run once per environment, therefore only one runtime is started
- Deploy many applications to it, therefore all the libraries are loaded into memory only once
- Services are loaded lazily, so unused parts reside on HDD, but no attempt to occupy RAM is made
- Curated package - well tested
- Used by many - most of the bugs discovered and fixed quickly
- Professional support available

Applications use databases, web services, REST services, dependency injection ... for every JSR, there is a place. In addition, the standard specifications and the implementations were made by better people than "average joe developer", which I also consider to be a plus. 

## Fatjars on scene

Fatjars did not appear out of nowhere in 2015 and later. In fact, Fatjars are with us for a long time. When creating "normal" Java SE applications, this means applications not dependent on Java EE standard libraries, there are often some libraries used. For the program to run, they must be either present on the host machine, or can be distributed in ( or besides) the Java archive (*.jar). This is a common practice and all the major build tools even have plugins for that.

There is however one type of fatjar, that is kind-of new, at least its popularity is growing in 2017. Personally, I call them **Enterprise Fatjars**, or shortly **EFJ**. The idea behind them is to bundle application server components into the jar itself. As a result, the application server "becomes a part" of a simple Java Archive. Running such application is then very simple, just use `java -jar applicationName.jar`. This way distribution has certain attributes perceived as advantages by some:

- Easy to start and stop, knowledge of an application server is not required
- Exactly the dependencies required are bundled and nothing more

I do not mention lower memory consumption on purpose, because current application servers already provide lazy loading. For one application server with one application in it, the Enterprise Fatjar can save about 20 MB of RAM. But with a second application running on the same environment, the application server wins clearly, since running another fatjar requires the whole machinery to be started in a separate JVM. However, there are also some other downsides:

- **Huge size** - both in-memory and on HDD
- Running one runtime per environment is no more possible. One enterprise app = one runtime
- Loading the application server components only once per machine is no longer possible
- Build may become dependent on fatjar solution, making it hard to migrate back
- Generally no fast redeployments without additional tools, adding complexity to the project

Enterprise Fatjars, as already mentioned, have pieces of application server inside. There is no magic and the functionality formerly bundled in the application server did not disapper, it is now only bundled inside.

## Spring as a special case

Special case is Spring Framework. They call their Enterprise Fatjar [Spring Boot](https://projects.spring.io/spring-boot/). But before Spring Boot came, Spring Framework itself was kind-of special. Spring does not use all the functionality present in Java EE application servers, but commonly uses only parts of Java EE specs. And that part is rather small. The way Spring works is simple - basic Java EE JSRs are taken (servlets for example) and there is big amount of non-standard functionality added to it. When typical Spring application started (was deployed into an application server), only a very small amount of the standard libraries was used. The rest was started from scratch with each Spring application deployed into the application server. For this reason, it was and still is popular to deploy Spring into so called servlet containers (like Tomcat). Containers, unlike servers, provided a much smaller subset of Java EE specs. In fact just enought for Spring to run and not much more. This makes perfect sense for Spring Framework developers.

All the libraries were commonly present in the Java Web Archive (.war). The resulting WARs were huge. Common size was 100+ megabytes for real-world projects, 200+ or even 500+ megabytes for extreme ones made by incompetent people. Such extremely big archives (500+) are no fault of Spring, these are just results of general incompetency of developers. It serves as a perfect example of what happens, if Average Joe Developer is given the chance to include all the dependencies on his/her own. It ends in a disaster. Therefore, tools like Spring Boot are necessary, because those tools come with the "opinion". In real world, the keyword "opinionated" is quite common and for me, it is a polite way of saying that it does what average developers in software houses are not generally capable of - keeps things tidy. In case of Spring, Boot really helped a by forcing tidiness.

And then, the **question arises**. Why is a common application server needed, when in fact 95% of the libraries is ran again and again with each new Spring application deployed ? The answer is obviously: It is not needed. So why bother ? Bundling the small but crucial pieces of Java EE specs Spring might require to run into the JAR does not make big harm. In fact, there is one upside to it. Different Spring applications carried different versions of the same libraries in history and there were classloading issues all the time when the applications were deployed to the very same application server running in the very same JVM. Fatjars solve this problem and many more, but add a little bit of overhead. I must say, the overhead is considered negligible by many and truth be told, RAM and HDD space are extremely cheap.

Spring is a special case and creating an enterprise fatjar does little harm with Spring, since the obvious damage may be done just by using it and the advantages of running one application server are gone. When I'm talking about damage being done, I only mean the use case where several applications can run on one machine, under one JVM easily. Also, the fatjar builds [need conversion](https://spring.io/guides/gs/convert-jar-to-war/) when developers want to get back to war packaging.

## Java EE and Enterprise Fatjars

In pure Java EE world, we provide Application servers as an opinionated bundle that is aimed to provide the necessary plumbing and should "just work". You can grab an all-in-one package application server, or you can build one on your own. Strong point is - the all-in-one package is the default way. In Java EE world, building the container part-by-part is commonly called "microcontainers". There are many of those:

- <a href="http://www.payara.fish/payara_micro" target="_blank">Payara Micro</a>
- <a href="http://wildfly-swarm.io/" target="_blank">WildFly Swarm</a>
- <a href="http://tomee.apache.org/apache-tomee.html" target="_blank">TomEE</a>
- <a href="https://developer.ibm.com/wasdev/websphere-liberty/" target="_blank">WebSphere Liberty</a>
- [KumuluzEE](https://ee.kumuluz.com/)
- [Meecrowave](http://openwebbeans.apache.org/meecrowave/index.html)

In reality, almost every application server out there [can do it](https://github.com/eclipse/microprofile-conference). The idea is always the same - to create one single executable jar with all the dependencies inside.

### Programmatic redeployments

Java EE fatjar solutions are very advanced in terms of possibilities. Nice API for deployment using pure Java code is one of the cool possibilities. Personally, I am yet to face the use case for this technique besides testig. For example, [WildFly Swarm](https://wildfly-swarm.gitbooks.io/wildfly-swarm-users-guide/content/getting-started/shrinkwrap.html) supports usage of ShrinkWrap not only to deploy, but to create the archives. Payara Micro [also supports](https://payara.gitbooks.io/payara-server/content/documentation/payara-micro/deploying/deploy-program-bootstrap.html) in-code deployment. And other cool stuff, like direct deployment from maven repositories. Or, deployment using a command line. 

When these ways of redeployments are used, some of the main disadvantages are eliminated:

- A single JVM per many deployment units is started (real memory saving)
- "Archive scanning" is active - exactly the libraries required by the application are loaded

A [payara example] looks as simple as this:
{% highlight java %}
import fish.payara.micro.BootstrapException;
import fish.payara.micro.PayaraMicro;
import java.io.File;

public class EmbeddedPayara 
{
    public static void main(String[] args) throws BootstrapException 
    {
        File file = new File("/home/user/example.war");
        PayaraMicro.getInstance().addDeploymentFile(file).bootStrap();
    }
}
{% endhighlight %}

Deployment using an application server jar is also possible. Multiple deployment units can be deployed at once. Without problems.

{% highlight java %}
java -jar payara-micro.jar --deploy /home/user/example1.war --deploy /home/user/example2.war
{% endhighlight %}


### Application-independent

One big plus of Java EE enterprise fatjars is the creation process. It is non-intrusive. The resulting JAR can be created as an additional step to the original build process. The application itself does not even know it is built as a part of fatjar solution. This leaves all the doors open for you as a developer - the option to deploy to standard application server is still there without any additional tweaks. Let's take [payara as an example](https://payara.gitbooks.io/payara-server/content/documentation/payara-micro/deploying/deploy-cmd-line.html). 

{% highlight java %}
java -jar payara-micro.jar --deploy /home/user/example.war --outputUberJar exampleUberJar.jar
{% endhighlight %}

This creates an "enterprise fatjar" using Payara Micro from the original WAR. No actions needed. It can be automated with Gradle or Maven easily. 

Lazy loading libraries into memory is standard for every microcontainer out there, including Payara. And like in case of Spring Boot, developers using WildFly Swarm can choose their own application server parts present in the resulting fatjar [using a generator](http://wildfly-swarm.io/generator/). Payara always includes "everything" currently.


## Horizontal Scaling & Docker
Nowadays, there are slightly different priorities than in the not so distant past. For a long time, arguably the trend is to make the applications stateless to enable potentially massive [horizontal scaling](https://blog.zhaw.ch/icclab/files/2014/07/cnaeval.pdf). The applications are now built as server-invariant. Spawning instances on-demand is the selling use case for many. Traditional approach of hosting apps directly on VMs suffers from several disadvantages, such as the overhead of a full OS and slow start-up time, a degree of vendor lock-in, and relatively coarse-grained units for scaling [(source)](http://ts.data61.csiro.au/publications/nictaabstracts/8901.pdf).

How is this in any way related with Java, Spring, Spring Boot, Java EE or Fatjars ? Spawning instances on demand as a part of horizontal scaling strategy leads to one observation:

> Every single application, or service in modern terms, has different requirements for scaling. Since every application/service is scaled independently, each instance spawned requires independent runtime environment.

A modern way to do this is **Docker**. Each application has its own Docker image with all the dependencies. Docker and its tooling [provides](https://docs.docker.com/engine/swarm/) service discovery and load balancing out of the box. This leads to less use for one of the big advantages of an application server - shared libraries loaded into memory just once for many applications. Some services are quick to serve many requests and therefore require less running instances. Some require more. When each application/service is spawned, a new Java EE application server as the runtime is also spawned. Putting multiple applications into one application server in this environment becomes an anti-pattern. However, few years ago, size used to be a serious topic, as [demonstrated by Arun Gupta](https://blogs.oracle.com/arungupta/entry/why_java_ee_6_is). Now it seems we do not care anymore. Or do we ?


### Docker kills the application server ?

The opposite is the truth. Application server is the materialization of many valuable ideas:

- Used by many. Bugs are quick to be revealed and fixed.
- Support available.
- Well documented.
- Standard-compliant. Well tested and certified.
- Many options - no vendor lock in.

And potentially many others. These advantages are valid for both microcontainers bundled in fat jars and for traditional application servers. In fact, fatjars do not impact these ideas. Simply put, microcontainers are just a curated subset of original features. Since Docker is used, there is no general need to create fat jars whatsoever. The fat jar has one advantage - only the required libraries are bundled, therefore HDD space is saved. This is especially true for Spring Boot or WildFly Swarm. This is not the case with memory, since every application server nowadays uses lazy loading of libraries. In fact, the disk space overhead of Docker itself is bigger than the difference between Payara and Payara Micro, or Wildfly Web profile vs Wildfly Swarm. 

| Solution        | HDD Space   | Memory drained | Code used |
| ------------- |:-------------:| -----:|
| Payara 171 Full | 119 MB | 48 mb | [GitHub JAX-RS sample](https://github.com/Pscheidl/jax-rs-tiny-sample) |
| WildFly 10.1.0 standalone |  139 MB    |  43 MB | [GitHub JAX-RS sample](https://github.com/Pscheidl/jax-rs-tiny-sample) |
| WildFly Swarm | 45 MB  |  38 MB | [GitHub Swarm Sample](https://github.com/Pscheidl/wildfly-swarm-rest-showcase) |
| Payara Micro | 62,6 MB | 37 MB | [GitHub JAX-RS sample](https://github.com/Pscheidl/jax-rs-tiny-sample) |
| Payara Micro Fatjar | 62,2 MB | 37 MB | [GitHub JAX-RS sample](https://github.com/Pscheidl/jax-rs-tiny-sample) |
| Spring Boot MVC REST | 15 MB | 28 MB | [GitHub Spring Rest example](https://github.com/Pscheidl/spring-rest-tiny-example)|
| TomEE Web profile | 34 MB | 17 MB | [GitHub JAX-RS sample](https://github.com/Pscheidl/jax-rs-tiny-sample) |

All memory measurements are done after forced garbage collection. Measured with VisualVM. Feel free to grab the code and reproduce the results. What was measured was no real use case. Usually, applications use much more than a single REST API endpoint without any transaction management, dependency injection, thread management, pooling, timers et cetera. From the results and from the content of resulting packages, it can be seen that Java EE application servers bundle more functionality out of the box. With Spring, there is less functionality in the very basic application, therefore less space on HDD and also in memory. But please note that there is for example no JPA support. This showcase however demonstrated the very basic overhead. 

The overhead of starting a full-blown Java EE application servers is exceptionally low compared to other solutions. The HDD space cost is negligible. The lazy loading seems to work very well. The difference between a fat jar and an application server with the very same application deployed is about 10 MB. 20 MB in worst case. Do we need to care about this in 2017 ? I do not think so. Especially considering the fact that there is no administration console in the "enterprise fatjars".

Using application servers inside Docker containers introduces negligible overhead, but allows to ship just the business logic without the need to fine-tune the dependencies every time the application changes. There is only one single dependency by default, and that is Java EE 7 API. Nothing more.

{% highlight xml %}
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>7.0</version>
            <scope>provided</scope>
        </dependency>
{% endhighlight %}

There are official Docker images for all the major players:
- [Payara](https://hub.docker.com/u/payara/) (all versions)
- [WildFly](https://hub.docker.com/r/jboss/wildfly/)
- [WebSphere Libery](https://hub.docker.com/_/websphere-liberty/)
- [TomEE](https://hub.docker.com/_/tomee/)
- [GlassFish](https://hub.docker.com/_/glassfish/)

And many others. Feel free to search DockHub on your own.

Creating a Dockerized Java EE application is then a few lines in Dockerfile. For example with WildFly.

{% highlight java %}
FROM jboss/wildfly

#WildFly administration port
EXPOSE 9990

#Copy the WAR to be deployed during container startup 
ADD /opt/jboss/wildfly/standalone/deployments sample.war

#Add admin user
RUN /opt/jboss/wildfly/bin/add-user.sh admin admin --silent
CMD ["/opt/jboss/wildfly/bin/standalone.sh", "-b", "0.0.0.0", "-bmanagement", "0.0.0.0"]
{% endhighlight %}

#### Fast redeployments with Java EE + Docker

Running application server has incredible redeployment times, because only the basic application with the business logic is changed. All the rest is running and does not need to be started agin. This is great for drop-in fast production redeployments and even better for development. Fatjar, without additional tooling, must be started over and over again with every change. For real-world applications, the time for the application to start might be tens of seconds, maybe more. However, if just the business logic is swapped, few (up to hundreds) of milliseconds is usually required and the changes can be seen instantly. And it can be done without additional tooling, only basic tools that come with the operating system are needed - copying a file. For example, Spring [introduces additional tooling](http://docs.spring.io/spring-boot/docs/current/reference/html/howto-hotswapping.html) to solve this fatjar issue. An issue Java EE thin wars never had, so there is nothing to solve.

With Java EE **fast redeployments are easy**. Thanks to [Docker volumes](https://docs.docker.com/engine/tutorials/dockervolumes/), the redeployment can be done in no time. Without additional tools. All the application servers support deployment by copying the archive into a specific deployment folder. With Docker, the deployment folder can mapped to a folder of the host filesystem. By placing the WAR into this specific folder, deployment happens. Rewriting the archive in that folder means redeployment. Easy and done instantly.

{% highlight java%}
docker run -v /home/pavel/deployments:/opt/jboss/wildfly/standalone/deployments jboss/wildfly
{% endhighlight %}
Of course, the /home/pavel/deployments is my example folder :) Because the server is already started and the war contains only business logic, on my laptop it takes < 200 ms to redeploy. Bigger WARs (3,5 MB JSF project ) takes < 1 sec. The redeployment can be automated by writing a simple script or making it part of the build process. It can be done in Ant, Maven, Gradle, IDEs also support it. It is still only a simple file copy. Point is, it is independent and no specialized tools are required. In Gradle, it can be achieved with a very simple task.

{% highlight groovy %}
task deployWar(type: Copy) {
    from war.archivePath
    into "${warDeploymentFolder}"
}
{% endhighlight %}


## Summary
Overall, enterprise fatjars in pure Java EE do not bring critical in-memory savings. On the other hand, they introduce big HDD overhead. This overhead has now however become a standard pattern - thanks to Docker and general shift to stateless, horizontally scalable applications. The presented solutions are derived from existing application servers and therefore present the same curated set of libraries. The whole fatjar functionality is added on top of Java EE standards without any interference.

A major finding is simple - **Docker is the way to pack Java EE applications**, not enterprise fatjars. The application server memory overhead is just a few more megabytes higher, but everything is there and there is only one dependency. Redeployments are really fast, therefore the development is easier. 

Some fatjar use cases, such as programmatic deployment, support for ShrinkWrap, console deployment and exact control of the JSRs included in the application is therefore something worth more attention. The ability to decide in-code which applications will be deployed and when might be just something you may need in several rare use cases. Spring is a different story. Fatjars are now Spring Framework's way to deliver a curated set of runtime-required libraries. In Spring ecosystem, it really helps and since there is little amount of Java EE specs required by Spring (if any ?), deploying to Java EE application servers makes little sense. I see Spring Boot as an obvious evolution of Spring. With all the pros and cons mentioned.

The path we now go is a path of little reuse, but it seems like we do not have to care anymore, because there are plenty of resources. And Java EE fits into this environment just fine :)

#### Resources

To explore more, following links are recommended

- [Thin WARs, Java EE 7, Docker and Productivity](https://youtu.be/5N4EUDhrkec) - Adam Bien
- [Docker for Java Developers](https://www.youtube.com/watch?v=IgJXYU3GOM4) - Arun Gupta
- [High Availability with Docker and Java EE](https://www.youtube.com/watch?v=kG_71pliL3o) - Elder Moraes & Andre Corvalho

#### Edits

3.4.2017: Added TomEE measurements and Meecrowave link
