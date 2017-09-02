---
title: "Logging in Java EE"
published: true
categories:
  - JEE-Tips
tags:
  - JEE
  - Logging
---
In year 2016, Logging in Java has it's own API with interchangeable implementation, of required. The API is called <a href="http://www.slf4j.org/" target="_blank">Simple Logging Facade for Java</a>, or simply SLF4J. I recommend using <a href="http://logback.qos.ch/" target="_blank">Logback</a> as a successor to Log4j. If nothing else, Logback is <a href="https://dzone.com/articles/which-one-faster-log4j-or" target="_blank">a tiny bit faster than</a> it's rivals. But with SLF4J, you can swap the implementation without any impact on the application.


### Dependencies

The logging face and it's implementation are required, resulting in two dependencies.

{% highlight xml %}
<!--        The Api to log with-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.21</version>
</dependency>
<!--        The actual logger implementation-->
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.1.7</version>
</dependency>
{% endhighlight %}

Or if you are using Gradle (a better tool), the dependencies format is following

{% highlight groovy %}
compile group: 'org.slf4j', name: 'slf4j-api', version: '1.7.21'
compile group: 'ch.qos.logback', name: 'logback-classic', version: '1.1.7'
{% endhighlight %}


### The code
A CDI producer has to be created. The producer's output is compliant with SLF4 - the producer logger is `org.slf4j.Logger`. SLF4J contains a class that automatically detects present implementations- `org.slf4j.LoggerFactory`. Calling it's getLogger methods instantiates the logger. If Logback is used, the actual implementation will be Logback logger.

A logger requires a name. It is widely used standard to name the logger with the same name as the class the logger's output comes from. If the Logger is injected, CDI provides a mechanism to find out the name of the class the instantiated logger is injected into. CDI will automatically inject `InjectionPoint` each time the producing method is called. From the `InjectionPoint` object, target bean's class can be obtained by calling `injectionPoint.getBean().getBeanClass()`. 

In practice, I can see loggers being manually constructed per each class. This is not only unnecessary, but also error-prone. Developers tend to copy code and forget old classname in newly created log. By using a CDI producer, each bean receives the right producer by simply injecting the logger.

{% highlight java %}
import javax.enterprise.inject.Produces;
import javax.enterprise.inject.spi.InjectionPoint;
import javax.inject.Named;
import javax.inject.Singleton;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;


@Named
@Singleton
public class LoggerProducer {

    @Produces
    public Logger produceLogger(InjectionPoint injectionPoint) {
        return LoggerFactory.getLogger(injectionPoint.getBean().getBeanClass());
    }

}
{% endhighlight %} 

Now, the logger can be injected into every CDI bean or an EJB present in the project. It's name will always be correct, change of implementation is simple and no refactoring on large systems is necessary. Simply inject the logger instance by ` @Inject private Logger logger;`. A trivial example shows how easy the usage is. 

{% highlight java %}
import javax.annotation.PostConstruct;
import javax.inject.Inject;
import javax.inject.Named;
import org.slf4j.Logger;


@Named
public class SimpleBean {

    @Inject
    private Logger logger;

    @PostConstruct
    private void postConstruct() {
        logger.debug("Initiated @PostConstruct method.");
    }

}
{% endhighlight %} 
