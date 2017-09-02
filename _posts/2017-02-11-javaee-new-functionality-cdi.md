---
title: "The source of new Java EE functionality: CDI and beyond"
published: true
categories:
  - JavaEE
tags:
  - JavaEE
  - CDI
---

Java EE is and should be primarily perceived as a set of well though-out standards. Does it mean that it provides mostly outdated stuff ? **No !** In fact, Java EE literally wants to be extended with the newest ideas ! Java EE is on it's own well crafted, however, there is an excellent built-in support for so called [Portable extensions](http://docs.jboss.org/cdi/spec/1.2/cdi-spec.html#spi). It allows for the new technology to be **plugged in easily**, without introducing any configuration or dependency mess for the developers. For the plugin authors, plugins are easy to write and can be very, very powerful. And there is already **a lot of functionality** to choose from. Because extensibility is also part of the standard.

Just remember, new, non-standard functionality does not always have to be a CDI extension. Often, it can be plugged right in.

## Portable extensions

Java EE ecosystem is rich and truly open. As an extension user, you can make use of work of many different companies and authors. As an author, you can plug your desired functionality directly into Java EE. In the following list, I am trying to demonstrate that Java EE world is in fact full of innovation. If you have a look at the source code of the projects listed below, you can see for yourself how easy it is to create a plugin.

### Databases

- [Apache Deltaspike Data](https://deltaspike.apache.org/documentation/data.html) - Like Spring Data ? This is better designed !
- [Spring Data](http://projects.spring.io/spring-data/) - Carries many heavyweight & proprietary Spring dependencies. Can be used if required.
- [JOOQ](https://www.jooq.org/) - This is not a CDI plugin, there is no need. But integrates with Java EE well.
- [MyBatis](https://github.com/mybatis/cdi)
- [CDI Query](https://github.com/ctpconsulting/query)

### Configuration
- [Apache Deltaspike Configuration](https://deltaspike.apache.org/index.html) - A very capable configuration library.
- [Apache Tamaya](http://tamaya.incubator.apache.org/)
- [CFG4J](http://www.cfg4j.org/)
- [CDI Properties](https://github.com/gonmarques/cdi-properties)
- [SoftwareMill configuration](https://github.com/softwaremill/softwaremill-common/tree/master/softwaremill-conf)

### Security
- [Deltaspike Security](https://deltaspike.apache.org/documentation/security.html)
- [Apache Shiro](https://shiro.apache.org/)
- [PAC4J](https://github.com/pac4j/jax-rs-pac4j)- JAX-RS security
- [PicketLink](http://picketlink.org/)
- [SoftwareMill security](https://github.com/softwaremill/softwaremill-common)
- [Keycloak](http://www.keycloak.org/)

### View
- [Vaadin](https://vaadin.com/docs/-/part/framework/advanced/advanced-cdi.html) - Well-known view framework is very clean to use with Java EE.
- [Ocelotds](http://ocelotds.org/) - The best and easiest communication way between java EE and javascript :)
- [Errai](http://erraiframework.org/)
- [VRaptor](http://www.vraptor.org/en/) - No MVC for Java EE ? This is a very common myth. Besides the new Ozark (MVC 1.0), there is VRaptor ! A clean and tidy MVC integration.
- [AngularBeans](http://bessemhmidi.github.io/AngularBeans/) - Like AngularJS ? Thanks to this project, Java EE has direct support for it.
- [Thymeleaf-CDI](https://github.com/inbuss/thymeleaf-cdi) - JSF are great, but if you are used to Thymeleaf, here you go. 
- [Thymeleaf-MVC](https://github.com/inbuss/thymeleaf-mvc) - Thymeleaf also already players with Java EE 8 MVC 1.0 :) There is also (JAX-RS integration](https://github.com/inbuss/thymeleaf-jersey)
- [Primefaces](http://primefaces.org/) - Represents the power of JSF. Primefaces is the ultimate JSF view library. Not only a set of components, but also themes, view utilities and many others. 
- [OmniFaces](http://showcase.omnifaces.org/)
- [BootsFaces](https://www.bootsfaces.net/)
- [Apache Wicket](https://github.com/42Lines/wicket-cdi)


### Uncategorized
- [Apache Camel](https://github.com/astefanutti/camel-cdi)
- [Metrics](https://github.com/astefanutti/metrics-cdi) - Metrics for your entrirprise applications
- [DeltaSpike monitoring](https://deltaspike.apache.org/addons.html) - This project has the same goals as Metrics
- [Arquillian](http://arquillian.org/) - Whole testing framework with bunch of CDI extensions. Absolutely unmatched testing platform.
- [JGlue Unit testing](http://jglue.org/cdi-unit-user-guide/) - Unit tests with CDI
- [Scripting extension](https://github.com/gunnarmorling/scripting-extension)
- [Method-level caching](https://github.com/logiquesistemas/cdi-cache-extension)
- [Quartz scheduler](https://github.com/Mark80/quartz-cdi) - Java EE has great scheduling inside, but if you want Quartz, you can have it.
- [Porcupine](https://github.com/AdamBien/porcupine) - Porcupine is the implementation of the bulkhead and handshaking patterns for Java EE 7
- [Breakr](https://github.com/AdamBien/breakr) - A Minimalistic Circuit Breaker for Java EE applications
- [Fabric8](https://fabric8.io/guide/cdi.html)
- [Assisted Injection](http://www.warski.org/blog/2010/10/di-and-oo-assisted-inject-in-cdi-weld/)
- [Paypal integration](https://github.com/softwaremill/softwaremill-common/tree/master/softwaremill-paypal)
- [Configure CDI beans in XML](https://github.com/rmannibucau/cdi-light-config) - If legacy classes are required to be part of dependency injection graph :)
- [Guice configuration for Java EE](https://github.com/avrecko/weld-guiceconfig) - Like Google Guice style of bean declaration ? There is a plugin for that too ! 
- [Dynamic messages](https://github.com/etecture/dynamic-messages)
- [Spring bridge](https://deltaspike.apache.org/addons.html) - If you have some heavyweight Spring application with a robust DI tree and want to @Inject into CDI, thanks to this bridge, you can.


There are many others to be found via Google, on GitHub and in many other places. The list goes on and on. For example, Java EE applications can also be rapidly created with [JBoss Forge](http://forge.jboss.org/) - this is also thanks to CDI and it's robust model.

## Become an author !

Writing CDI extensions is easy. By contributing directly into Java EE, you make sure your code is not tied to a proprietary platform and can be used by widest enterprise audience - Java EE developers. With CDI extensions, you can do **just about anything** ! Just remember, Java EE is very well designed and not every problem needs to be represented as an extension - this further improves the easiness of coding with Java EE. There are more capable and informed people you can visit and have a look at their great guides about what can be achieved with Java EE + CDI and how to achieve it.

- [CDI Extension guide](http://www.next-presso.com/2017/02/nobody-expects-the-cdi-portable-extensions/) by Antoine Sabot-Durand
- [CDI 1.2 Specification](http://docs.jboss.org/cdi/spec/1.2/cdi-spec.html#spi) - By Gaving King and many others. There is a newever version on it's way, so beware !
- [Write your own CDI Extension](https://www.javacodegeeks.com/2014/02/tutorial-writing-your-own-cdi-extension.html) by Peter Daum
- [CDI extensions by example](http://www.byteslounge.com/tutorials/java-ee-cdi-extension-example) - by Gonçalo Marques
- [With CDI Extensions, You Can Build Your Own Java 7](https://dzone.com/articles/cdi-extensions-you-can-build) - by Mitch Pronschinske

## CDI - the cool guy 

As you can see, Java EE is a very rich environment. On the shoulders of giants (standards made by very capable people), there are many innovative project that play nicely with Java EE. There is only one thing to be changed soon (hopefuly) - there is no central place to inform newcomers about Java EEs capabilities. Java EE represent a common effort of many companies and individuals. Having a common place to inform about Java EE's capabilities would be nice.

- [Type qualifiers](http://docs.jboss.org/cdi/spec/1.2/cdi-spec.html#qualifiers) - In Java EE, we do not use Strings for qualifiers :)
- [Events](http://docs.jboss.org/cdi/spec/1.2/cdi-spec.html#events)
- [Decorators](http://docs.jboss.org/cdi/spec/1.2/cdi-spec.html#decorators) - [explained](http://www.mastertheboss.com/jboss-frameworks/cdi/interceptors-and-decorators-tutorial)
- [Interceptors](http://docs.jboss.org/cdi/spec/1.2/cdi-spec.html#interceptors)

And many, many others. If you want to get to know more about Java EE, CDI and it's capabilities, I always recommend [Beginning Java EE 7](http://www.apress.com/us/book/9781430246268) by Antonio Goncalves. It is the book I recommend to both newcomers are experienced developers. It is a book of great quality, very nicely written. And there is also a "nice" [story behind this book](https://antoniogoncalves.org/2014/09/16/the-uncensored-java-ee-7-book/).
