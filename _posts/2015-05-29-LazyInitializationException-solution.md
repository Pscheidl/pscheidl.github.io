---
title: "How to solve Hibernate's LazyInitializationException"
published: true
categories:
  - JEE-Tips
tags:
  - JEE
  - JPA
---


If Hibernate is used as a JPA implementation inside the application server (JBoss, WildFly), LazyInitializationException is a result of Hibernate's incompatibility with JPA standard. Hibernate requires transaction, yet there is none and it fails automatically creating a new one. Transactions starts, data are selected from underlying database and an object is returned. But when the application tries to access object's lazily initialized property, LazyInitializationException occurs.


This typically happens when you retrieve an object from database and store it in JSF bean. When JSF tries to fill the view with appropriate values, then it calls the getter of that object retrieved from database. But the transaction has already ended.

### The solution

**a)** Add this one line of code into your persistence.xml.

{% highlight xml %}
<property name="hibernate.enable_lazy_load_no_trans" value="true"/>
{% endhighlight %}

**Done**. Hibernate will load object's attributes without requiring transaction to be started.

Your persistance.xml can now look similar to this example code:

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.1" xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_1.xsd">
    <persistence-unit name="mysqlPersistenceUnit" transaction-type="JTA">
        <provider>org.hibernate.ejb.HibernatePersistence</provider>
        <jta-data-source>java:/jdbc/mySQL</jta-data-source>
        <exclude-unlisted-classes>false</exclude-unlisted-classes>
        <shared-cache-mode>NONE</shared-cache-mode>
        <properties>
            <property name="hibernate.enable_lazy_load_no_trans" value="true"/>
        </properties>
    </persistence-unit>
</persistence>
{% endhighlight %}

**b)** Use JPA's reference implementation called EclipseLink instead. Nowadays, it just works better. For example, probably the best application server of today, <a href="http://www.payara.fish/" target="_blank">Payara</a>, uses EclipseLink. 