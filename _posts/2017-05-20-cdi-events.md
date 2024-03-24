---
title: "CDI Events overview"
published: true
author: pavel_pscheidl
categories:
  - JavaEE
tags:
  - JavaEE
  - CDI
---

Java Enterprise Edition has many features that really stand out. One of the best is **event mechanism**, which is part of the [Contexts & Dependency Injection for Java](https://docs.jboss.org/cdi/spec/2.0/cdi-spec.html) specification. Events are present in Java EE for a long time. Design of the events mechanism is extremely clean and learning how to use events is therefore very simple. This overview is aimed at developers who are not familiar with the event mechanism and wish to get to know about the basics. Advanced features part of CDI 2.0 like [Asynchronous events](https://docs.jboss.org/cdi/spec/2.0/cdi-spec.html#firing_events_asynchronously) are not covered. You will learn to

- Fire specific event,
- Use event qualifiers,
- Observe events fired during transactions,
- Configure even observer bean instantiation.


At the end of this article, you will find instruction for a *quick-to-run* sample application available on [GitHub](https://github.com/Pscheidl/cdi-events-playground), demonstrating capabilities of CDI events.

## Simple events

The goal is to fire an event in one component of the application and listen to it anywhere else. This allows the application components to be more loosely coupled. Whenever an event is fired, all the subscribers/listeners receive it. Java EE (CDI) internally manages a list of all listeners and decides which one to call when. Automatically, without any configuration. 

### Firing an event

Firing an event is as simple as injecting a `javax.enterprise.event.Event<T>` into a bean and invoking its `fire(T t)` method.

```java
@Named
public class EventSource {

    @Inject
    private Event<String> simpleMessageEvent;

    public void fireEvent(){
    	simpleMessageEvent.fire("Hello");
    }

}
```

The class  `Event<T>` from `javax.enterprise.event` package is instantiated and managed automatically by CDI. As you have probably already noticed, there is one generic type `T` inside the `Event<T>` class. This is the **type of message** held inside each and every event fired. Only methods observing the same type will receive the event. The generic type can of course be any Java type. 


### Listening to an event

In order to observe events application-wide, two steps are required:

- Create a method accepting the desired type of event as an argument.
- Annotate the method's argument with `javax.enterprise.event.Observes` annotation.

```java
@Named
public class EventObserver {

    public void observeEvent(@Observes String message){
    	System.out.println(message);
    }

}
```

The number of methods listening to certain type of event, where type is the generic parameter `T` defined inside the injected `Event<T>`, is **not limited**. Of course, the method must be present inside any kind of Java EE Bean (CDI, EJB).



## Event qualifiers

A common use case is to categorize or in general differentiate events from each other. Event observers can then decide which events they want to listen to. This is done by means of a [CDI Qualifier](https://docs.jboss.org/cdi/spec/2.0/cdi-spec.html#qualifiers). A qualifier is in fact a fancy annotation. 



```java
@Qualifier
@Retention(RUNTIME)
@Target({METHOD, FIELD, PARAMETER, TYPE})
public @interface Important {
}
```

The resulting `@Important` annotation is recognized as a qualifier by placing the `@Qualifier` annotation on it. Name of the annotation is arbitraty. You can you just about anything that is allowed for Java annotations.


_There is one strong reason annotations are used as qualifiers: CDI is well thought out and leverages type safety. Using built-in Java types instead of e.g. prehistoric Strings has many benefits for the programmer. One of the strongest reasons are compile-time checks - developers are no longer forced to waste time running the application with each change to check if everything is wired correctly. The compiler does the check naturally._

### Firing an event with a qualifier

The mechanism of firing an event remains. Only the created qualifier `@Important` is used to annotate the injected `Event<String>`. Nothing more is required. The `fire()` method will automatically let the observers know the message is `@Important`. There is no work for the developer.

```java
@Named
public class SelectedEventSource {

    @Inject
    @Important
    private Event<String> importantMessageEvent;

    public void fireEvent(){
    	simpleMessageEvent.fire("Hello");
    }

}
```

There is also an alternative way of firing an event with a qualifier demonstrated in the [sample application], using the `select()` method of an `Event<T>`.


### Observing only selected events

Using the `@Important` qualifier, the event is internally marked as `@Important`. But this does not stop general observers from receiving the event. Event qualifiers are instead used to limit the mass of the set of event types an observer receives. This limitation is done by simply using the qualifier `@Important` together with the `@Observes annotation`.


```java
@Named
public class ImportantEventObserver {

    public void observeImportantMessage(@Observes @Important String message){
    	System.out.println(message);
    }

}
```

This methods will receive only events marked as `@Important` with type `String` and no other events. 

## Transations and Events

Java EE has a concept of [tranasctions](https://docs.oracle.com/javaee/6/tutorial/doc/bncih.html) for a long time. Transactions may fail and be rolled backed or may be completed sucessfully. During transactions, events may be fired. But what if the event observes wants to receive the event only if the transaction ended up in a certain state, for example successful completion ?

This is very easy to achieve with CDI. The `@Observes annotation` has a field named `during`. This field accepts values from the `javax.enterprise.event.TransactionPhase` enumeration.

```java
@Named
public class TransactionEventObserver {

    public void observeImportantMessage(@Observes(during = TransactionPhase.AFTER_SUCCESS) String message){
    	System.out.println(message);
    }

}
```

This way, the observer will be notified only after the transaction ends successfully. There are 5 possible states defined by `TransactionPhase`. The default value is `TransactionPhase.IN_PROGRESS` - the event is delivered immediately after it was fired.

```java
public enum TransactionPhase {
    IN_PROGRESS,
    BEFORE_COMPLETION,
    AFTER_COMPLETION,
    AFTER_FAILURE,
    AFTER_SUCCESS
}
```

### Event reception

In a situation when an event is fired, but the observer resides in a bean with no living instance, there are two scenarios possible.

1. CDI creates the instance and delivers the event to the observer (`Reception.ALWAYS`)
1. The event is not delivered (`Reception.IF_EXISTS`)


This is configured easily per each observer by means of a `notifyObserver` attribute of the `@Observes` annotation. The attribute accepts values from `javax.enterprise.event.Reception` enumeration. There are only two possible values, `Reception.ALWAYS` and `Reception.IF_EXISTS`. By default, when the bean instance is not created, CDI will create it and deliver the event. This corresponds with the first option. The `Reception.ALWAYS` is used as a default value and can be omitted. In the following example, only the `@Observes` annotation could be used.

```java
@Named
public class EventObserver {
	// Explicit declaration of the default value - can be omitted
    public void observeEvent(@Observes(notifyObserver = Reception.ALWAYS) String message){
    	System.out.println(message);
    }

}
```

If the value is changed to `Reception.IF_EXISTS`, CDI will delivered the event only to the currently instantiated beans.

```java
@Named
public class EventObserver {
	// If there is no bean instantiated, create one
    public void observeEvent(@Observes(notifyObserver = Reception.IF_EXISTS) String message){
    	System.out.println(message);
    }

}
```

## Sample project

There is a sample application on [GitHub](https://github.com/Pscheidl/cdi-events-playground) created to demonstrate the capabilities of CDI events. Starting the application is easy and no configuration or manual downloads are required, only **Maven's presence** is needed.

The most **simple and fastest** way to do it is to execute **maven goal** `wildfly-swarm:run`. On the first run, the WildFly Swarm microcontainer will be downloaded. The application will be scanned and only the parts required to run the application will be downloaded automatically - a few megabates. In the end, Java EE is about choice, thus feel free to choose whichever way of running the Java EE sample application fits you the best.

The application has clickable GUI to play with.