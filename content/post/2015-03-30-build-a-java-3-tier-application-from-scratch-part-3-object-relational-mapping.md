---
id: 3148
title: 'Build a Java 3-tier application from scratch – Part 3: Object-relational mapping'
date: 2015-03-30T16:41:24+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=3148
permalink: /2015/03/30/build-a-java-3-tier-application-from-scratch-part-3-object-relational-mapping/
dsq_thread_id:
  - "3639842548"
image: /wp-content/uploads/2014/10/Java-logo.jpg
categories:
  - Java
tags:
  - 3-tier
  - access
  - application
  - authentication
  - authenticator
  - client
  - controller
  - create
  - crud
  - dao
  - data
  - delete
  - development
  - faker
  - filter
  - get
  - java
  - jax-rs
  - json
  - layers
  - method
  - object
  - request
  - rest
  - restful
  - seeder
  - server
  - update
  - webservice
---
Welcome to third part of my 3-tier application tutorial. Within this and the next part we are going to develope simple webservice that communicates with the database and maps Java objects to data tables.
We will create a controller that communicates with our MySQL database using the EclipseLink ORM to abstract this process.

Here's a picture of what we want to achieve. A simple webservice that's serves depending on the url an array of json data.

<img src="https://janikvonrotz.ch/wp-content/uploads/2015/03/Java-3-tier-webservice.png" alt="Java 3-tier webservice" width="717" height="251" class="aligncenter size-full wp-image-3150" />
<!--more-->

Let's get started with the file structure. Create all the files, packages and directories as showed below.

<img src="https://janikvonrotz.ch/wp-content/uploads/2015/03/Webservice-Eclipse-Filestructure.png" alt="Webservice Eclipse Filestructure" width="279" height="391" class="aligncenter size-full wp-image-3163" />

In this tutorial I won't show you how can install and configure a MySQL server. It's expected that you already have a running server and a prepared database.

# EclipseLink ORM

Instead of the common property file our application stores settings in a json file.

* Update the `applicaton.json` file.

**application.json**

[code]
{
  &quot;javax&quot;: {
    &quot;persistence&quot;: {
      &quot;jdbc&quot;: {
        &quot;driver&quot;: &quot;com.mysql.jdbc.Driver&quot;,
        &quot;url&quot;: &quot;jdbc:mysql://localhost/issuemanager&quot;,
        &quot;user&quot;: &quot;issuemanager&quot;,
        &quot;password&quot;: &quot;issuemanager&quot;
      }
    }
  },
  &quot;eclipselink&quot;: {
    &quot;logging&quot;: {
      &quot;level&quot;: &quot;off&quot;
    },
    &quot;ddl-generation&quot;: {
    	&quot;value&quot;: &quot;create-tables&quot;,
    	&quot;output-mode&quot;: &quot;database&quot;
	}
  }
}
[/code]

* Make sure the update the jdbc url, user and password property.

* As next we will create the `EntityFactory` which allows us to access the database with plain simple java objects.

**EclipseLink.java**

[code lang="java"]
package ch.issueman.webservice;

import java.util.HashMap;
import java.util.Map;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;

import com.typesafe.config.Config;
import com.typesafe.config.ConfigFactory;

public class EclipseLink {

	private static EntityManagerFactory emf = null;
	
	static {
		try {
			Config config = ConfigFactory.load();
			Map&lt;String, String&gt; properties = new HashMap&lt;String, String&gt;();
			properties.put(&quot;javax.persistence.jdbc.driver&quot;, config.getString(&quot;javax.persistence.jdbc.driver&quot;));
			properties.put(&quot;javax.persistence.jdbc.url&quot;, config.getString(&quot;javax.persistence.jdbc.url&quot;));
			properties.put(&quot;javax.persistence.jdbc.user&quot;, config.getString(&quot;javax.persistence.jdbc.user&quot;));
			properties.put(&quot;javax.persistence.jdbc.password&quot;, config.getString(&quot;javax.persistence.jdbc.password&quot;));
			properties.put(&quot;eclipselink.ddl-generation.output-mode&quot;, config.getString(&quot;eclipselink.ddl-generation.output-mode&quot;));
			properties.put(&quot;eclipselink.logging.level&quot;, config.getString(&quot;eclipselink.logging.level&quot;));
			properties.put(&quot;eclipselink.ddl-generation&quot;, config.getString(&quot;eclipselink.ddl-generation.value&quot;));
						
			emf = Persistence.createEntityManagerFactory(&quot;issue-manager&quot;, properties);
		} catch (Throwable e) {
		}
	}

	public static EntityManagerFactory getEntityManagerFactory() {
		return emf;
	}

	public static EntityManager getEntityManager() {
		return emf.createEntityManager();
	}
}
[/code]

Based on our `persistence.xml` file and configure with the `application.json` file this class stores a static instance of the EclipseLink entity factory. 

* As next update the persistence configuration file.

**persistence.xml**

[code lang="xml"]
&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot;?&gt;
&lt;persistence version=&quot;2.0&quot; xmlns=&quot;http://java.sun.com/xml/ns/persistence&quot; xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot; xsi:schemaLocation=&quot;http://java.sun.com/xml/ns/persistence http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd&quot;&gt;
	&lt;persistence-unit name=&quot;issue-manager&quot; transaction-type=&quot;RESOURCE_LOCAL&quot;&gt;

		&lt;provider&gt;org.eclipse.persistence.jpa.PersistenceProvider&lt;/provider&gt;

        &lt;class&gt;ch.issueman.common.Person&lt;/class&gt;
        &lt;class&gt;ch.issueman.common.User&lt;/class&gt;
        &lt;class&gt;ch.issueman.common.Employer&lt;/class&gt;
        &lt;class&gt;ch.issueman.common.Project&lt;/class&gt;
        &lt;class&gt;ch.issueman.common.Comment&lt;/class&gt;
        
		&lt;exclude-unlisted-classes&gt;false&lt;/exclude-unlisted-classes&gt;
				
	&lt;/persistence-unit&gt;
	
&lt;/persistence&gt;
[/code]

You might come along other persistence files with far more properties, but that's actually all we need as we use a different (better) approach to configure EclipseLink.

When working with the MVC model you have to implement a controller to communicate with the database and to apply business logic. So does our application. Based on the DAO interface our controller uses the entity manager does common CRUD actions with the models. As we don't want to write a controller for each model our controller uses generic types. This might confuse you at the beginning, but it will be more obvious when we instance the model controllers.

* Now update the controller class.

**Controller.java**

[code lang="java"]
package ch.issueman.webservice;

import java.io.Serializable;
import java.util.List;

import javax.persistence.EntityManager;
import javax.persistence.TypedQuery;

import ch.issueman.common.DAO;

public class Controller&lt;T, Id extends Serializable&gt; implements DAO&lt;T, Id&gt; {

	private EntityManager em = null;
	private final Class&lt;T&gt; clazz;

	public Controller(Class&lt;T&gt; clazz) {
		this.clazz = clazz;
		em = EclipseLink.getEntityManager();
	}

	public void persist(T t) {
		em = EclipseLink.getEntityManager();
		em.getTransaction().begin();
		em.persist(t);
		em.getTransaction().commit();
		em.close();
	}

	public T getById(Id id) {
		em = EclipseLink.getEntityManager();
		return em.find(clazz, id);
	}

	public List&lt;T&gt; getAll() {
		em = EclipseLink.getEntityManager();
		return (List&lt;T&gt;) ((TypedQuery&lt;T&gt;) em.createQuery(&quot;SELECT t FROM &quot; + clazz.getSimpleName() + &quot; t&quot;, clazz)).getResultList();
	}

	public void update(T t) {
		em = EclipseLink.getEntityManager();
		em.getTransaction().begin();
		em.merge(t);
		em.getTransaction().commit();
		em.close();
	}

	public void delete(T t) {
		em = EclipseLink.getEntityManager();
		em.getTransaction().begin();
		em.remove(em.merge(t));
		em.getTransaction().commit();
		em.close();
	}

	public void deleteAll() {
		em = EclipseLink.getEntityManager();
		em.getTransaction().begin();
		em.createQuery(&quot;DELETE FROM &quot; + clazz.getSimpleName() + &quot; t&quot;).executeUpdate();
		em.getTransaction().commit();
		em.close();
	}
}
[/code]

Wow, that was a lot of code, confused yet? I will explain the most important code lines.

* The clazz variable stores the model type. This is actually a workaround so we can access the model class properties such as its name.
* Every action that alters data (persist, delete, update) must call the Transaction manager in order to commit the changes.
* The query in the in the `getAll()` and `deleteAll()` method is not SQL, it's JPQL (Java Persistence Query Language).

Very well, now we've set up everything to use the advanced features of an ORM. Next we are going to configure the webservice.

# Update

* 2015-01-04: Improved the controller by shorten the transactional methods.

# Links

* [Part 1: Introduction and project setup](https://janikvonrotz.ch/2015/03/15/build-a-java-3-tier-application-from-scratch-part-1-introduction-and-project-setup/)
* [Part 2: Model setup](https://janikvonrotz.ch/2015/03/28/build-a-java-3-tier-application-from-scratch-part-2-model-setup/)
* [Part 3: Object-relational mapping](https://janikvonrotz.ch/2015/03/30/build-a-java-3-tier-application-from-scratch-part-3-object-relational-mapping/)
* [Part 4: Webservice](https://janikvonrotz.ch/2015/03/31/build-a-java-3-tier-application-from-scratch-part-4-webservice/)
* [Part 5: Client controller](https://janikvonrotz.ch/2015/04/01/build-a-java-3-tier-application-from-scratch-part-5-client-controller/)
* [Part 6: Client view](https://janikvonrotz.ch/2015/04/02/build-a-java-3-tier-application-from-scratch-part-6-client-view/)
# Source

[Feedback from Reddit on this post](http://www.reddit.com/r/java/comments/3119as/build_a_java_3tier_application_from_scratch_part/)