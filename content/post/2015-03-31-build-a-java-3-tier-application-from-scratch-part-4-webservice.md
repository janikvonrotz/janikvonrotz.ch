---
id: 3177
title: 'Build a Java 3-tier application from scratch – Part 4: Webservice'
date: 2015-03-31T08:05:04+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=3177
permalink: /2015/03/31/build-a-java-3-tier-application-from-scratch-part-4-webservice/
dsq_thread_id:
  - "3641869503"
image: /wp-content/uploads/2014/10/Java-logo.jpg
categories:
  - Java
tags:
  - configuration
  - controller
  - crud
  - java
  - jax-rs
  - jetty
  - json
  - mapping
  - model
  - object
  - orm
  - path
  - relational
  - rest
  - resteasy
  - route
  - tomcat
  - url
  - view
  - web
  - webservice
  - xml
---
Our ORM is working so we are ready to create our JAX-RS restful webservice and implement a custom authentication mechanism. In addition I'll provide you a class to insert dummy data into the database, so you don't have to do it manually.
<!--more-->
# RESTeasy servlet

RESTeasy is our JAX-RS provider and client component. In this part we will conigure a RESTeasy servlet and add some routes to handle by the webservice.

* Update the webservice configuration file.

[code lang="xml"]
&lt;web-app id=&quot;WebApp_ID&quot; version=&quot;2.4&quot;
	xmlns=&quot;http://java.sun.com/xml/ns/j2ee&quot; xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
	xsi:schemaLocation=&quot;http://java.sun.com/xml/ns/j2ee 
	http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd&quot;&gt;
	&lt;display-name&gt;Restful Web Application&lt;/display-name&gt;

	&lt;context-param&gt;
		&lt;param-name&gt;resteasy.resources&lt;/param-name&gt;
		&lt;param-value&gt;ch.issueman.webservice.Route&lt;/param-value&gt;
	&lt;/context-param&gt;

	&lt;context-param&gt;
		&lt;param-name&gt;resteasy.providers&lt;/param-name&gt;
		&lt;param-value&gt;ch.issueman.webservice.Authenticator&lt;/param-value&gt;
	&lt;/context-param&gt;

	&lt;servlet&gt;
		&lt;servlet-name&gt;resteasy-servlet&lt;/servlet-name&gt;
		&lt;servlet-class&gt;
			org.jboss.resteasy.plugins.server.servlet.HttpServletDispatcher
		&lt;/servlet-class&gt;
	&lt;/servlet&gt;

	&lt;servlet-mapping&gt;
		&lt;servlet-name&gt;resteasy-servlet&lt;/servlet-name&gt;
		&lt;url-pattern&gt;/*&lt;/url-pattern&gt;
	&lt;/servlet-mapping&gt;

&lt;/web-app&gt;
[/code]

The resources class mapps actions to url paths, a provider hooks up into every request done by a client such as authentication or access filtering.

Our Route class file has the most lines of code. But don't be anxious, it contains a lot of repetetive configurations. For every model we have to define the CRUD methods. A DRY (don't repeat yourself) approach is very difficult here, as a lot is going on behind the scenes. I only could stack the `get` methods together.
That means 5 models * 3 CUD-methods + 2 R-methods + 1 login method = 18 path actions for our webservice.

* Update the Route class.

**Route.java**

[code lang="java"]
package ch.issueman.webservice;

import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

import javax.annotation.security.PermitAll;
import javax.annotation.security.RolesAllowed;
import javax.ws.rs.Consumes;
import javax.ws.rs.DELETE;
import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.PUT;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;
import javax.ws.rs.core.Response.Status;

import ch.issueman.common.Employer;
import ch.issueman.common.Model;
import ch.issueman.common.Person;
import ch.issueman.common.User;
import ch.issueman.common.Project;
import ch.issueman.common.Comment;
import ch.issueman.common.DAO;

@Path(&quot;/&quot;)
public class Route{
	
	private Map &lt;String, DAO&lt;?, Integer&gt;&gt; hm = new HashMap&lt;String, DAO&lt;?, Integer&gt;&gt;();

	public Route(){
		hm.put(&quot;person&quot;, new Controller&lt;Person, Integer&gt;(Person.class));
		hm.put(&quot;user&quot;, new Controller&lt;User, Integer&gt;(User.class));
		hm.put(&quot;employer&quot;, new Controller&lt;Employer, Integer&gt;(Employer.class));
		hm.put(&quot;project&quot;, new Controller&lt;Project, Integer&gt;(Project.class));
		hm.put(&quot;comment&quot;, new Controller&lt;Comment, Integer&gt;(Comment.class));	
	}
	
	@RolesAllowed(&quot;Administrator&quot;)
	@GET
	@Path(&quot;{entity}/{id}&quot;)
	@Produces(MediaType.APPLICATION_JSON)
	public Model getEntityById(@PathParam(&quot;entity&quot;) String entity, @PathParam(&quot;id&quot;) int id) {
		return (Model) hm.get(entity).getById(id);
	} 	
	
	@PermitAll
	@SuppressWarnings(&quot;unchecked&quot;)
	@GET
	@Path(&quot;{entity}&quot;)
	@Produces(MediaType.APPLICATION_JSON)
	public List&lt;Model&gt; getAll(@PathParam(&quot;entity&quot;) String entity) {
		return (List&lt;Model&gt;) hm.get(entity).getAll();
	}	
	
	@PermitAll
	@SuppressWarnings({ &quot;unchecked&quot; })
	@POST
	@Path(&quot;login&quot;)
	@Consumes(MediaType.APPLICATION_JSON)
	@Produces(MediaType.APPLICATION_JSON)
	public Response login(User user) {
		
		List&lt;User&gt; users = ((List&lt;User&gt;) hm.get(&quot;user&quot;).getAll()).stream()
				.filter(p -&gt; p.getEmail().equals(user.getEmail()))
				.filter(p -&gt; p.getPassword().equals(user.getPassword()))
				.collect(Collectors.toList());
		
		if(users.size() == 1){
			return Response.status(Status.OK).entity(users.get(0)).build();
		}else{
			return Response.status(Status.UNAUTHORIZED).build();
		}
	}
	
	/**
	 * Person
	 */
	@RolesAllowed(&quot;Administrator&quot;)
	@SuppressWarnings({ &quot;unchecked&quot;, &quot;rawtypes&quot; })
	@DELETE
	@Path(&quot;person/{id}&quot;)
	@Consumes(MediaType.APPLICATION_JSON)
	public Response deletePerson(@PathParam(&quot;id&quot;) int id) {
		((DAO) hm.get(&quot;person&quot;)).delete(((DAO) hm.get(&quot;person&quot;)).getById(id));
		return Response.status(Status.OK).entity(&quot;Person deleted&quot;).build();
	}		
	@RolesAllowed(&quot;Administrator&quot;)
	@SuppressWarnings({ &quot;unchecked&quot;, &quot;rawtypes&quot; })
	@PUT
	@Path(&quot;person&quot;)
	@Consumes(MediaType.APPLICATION_JSON)
	public Response updatePerson(Person t) {
		((DAO) hm.get(&quot;person&quot;)).update(t);
		return Response.status(Status.OK).entity(&quot;Person updated&quot;).build();
	}		
	@RolesAllowed(&quot;Administrator&quot;)
	@SuppressWarnings({ &quot;unchecked&quot;, &quot;rawtypes&quot; })
	@POST
	@Path(&quot;person&quot;)
	@Consumes(MediaType.APPLICATION_JSON)
	public Response persistPerson(Person t) {
		((DAO) hm.get(&quot;person&quot;)).persist(t);
		return Response.status(Status.OK).entity(&quot;Person added&quot;).build();
	}
		
	/**
	 * Employer
	 */
	@RolesAllowed(&quot;Administrator&quot;)
	@SuppressWarnings({ &quot;unchecked&quot;, &quot;rawtypes&quot; })
	@DELETE
	@Path(&quot;employer/{id}&quot;)
	@Consumes(MediaType.APPLICATION_JSON)
	public Response deleteEmployer(@PathParam(&quot;id&quot;) int id) {
		((DAO) hm.get(&quot;employer&quot;)).delete(((DAO) hm.get(&quot;employer&quot;)).getById(id));
		return Response.status(Status.OK).entity(&quot;Employer deleted&quot;).build();
	}		
	@RolesAllowed(&quot;Administrator&quot;)
	@SuppressWarnings({ &quot;unchecked&quot;, &quot;rawtypes&quot; })
	@PUT
	@Path(&quot;employer&quot;)
	@Consumes(MediaType.APPLICATION_JSON)
	public Response updateEmployer(Employer t) {
		((DAO) hm.get(&quot;employer&quot;)).update(t);
		return Response.status(Status.OK).entity(&quot;Employer updated&quot;).build();
	}		
	@RolesAllowed(&quot;Administrator&quot;)
	@SuppressWarnings({ &quot;unchecked&quot;, &quot;rawtypes&quot; })
	@POST
	@Path(&quot;employer&quot;)
	@Consumes(MediaType.APPLICATION_JSON)
	public Response persistEmployer(Employer t) {
		((DAO) hm.get(&quot;employer&quot;)).persist(t);
		return Response.status(Status.OK).entity(&quot;Employer added&quot;).build();
	}
	
	/**
	 * User
	 */
	@RolesAllowed(&quot;Administrator&quot;)
	@SuppressWarnings({ &quot;unchecked&quot;, &quot;rawtypes&quot; })
	@DELETE
	@Path(&quot;user/{id}&quot;)
	@Consumes(MediaType.APPLICATION_JSON)
	public Response deleteUser(@PathParam(&quot;id&quot;) int id) {
		((DAO) hm.get(&quot;user&quot;)).delete(((DAO) hm.get(&quot;user&quot;)).getById(id));
		return Response.status(Status.OK).entity(&quot;User deleted&quot;).build();
	}		
	@RolesAllowed(&quot;Administrator&quot;)
	@SuppressWarnings({ &quot;unchecked&quot;, &quot;rawtypes&quot; })
	@PUT
	@Path(&quot;user&quot;)
	@Consumes(MediaType.APPLICATION_JSON)
	public Response updateUser(User t) {
		((DAO) hm.get(&quot;user&quot;)).update(t);
		return Response.status(Status.OK).entity(&quot;User updated&quot;).build();
	}		
	@RolesAllowed(&quot;Administrator&quot;)
	@SuppressWarnings({ &quot;unchecked&quot;, &quot;rawtypes&quot; })
	@POST
	@Path(&quot;user&quot;)
	@Consumes(MediaType.APPLICATION_JSON)
	public Response persistUser(User t) {
		((DAO) hm.get(&quot;user&quot;)).persist(t);
		return Response.status(Status.OK).entity(&quot;User added&quot;).build();
	}
	
	/**
	 * Project
	 */
	@RolesAllowed(&quot;Administrator&quot;)
	@SuppressWarnings({ &quot;unchecked&quot;, &quot;rawtypes&quot; })
	@DELETE
	@Path(&quot;project/{id}&quot;)
	@Consumes(MediaType.APPLICATION_JSON)
	public Response deleteProject(@PathParam(&quot;id&quot;) int id) {
		((DAO) hm.get(&quot;project&quot;)).delete(((DAO) hm.get(&quot;project&quot;)).getById(id));
		return Response.status(Status.OK).entity(&quot;Project deleted&quot;).build();
	}		
	@RolesAllowed(&quot;Administrator&quot;)
	@SuppressWarnings({ &quot;unchecked&quot;, &quot;rawtypes&quot; })
	@PUT
	@Path(&quot;project&quot;)
	@Consumes(MediaType.APPLICATION_JSON)
	public Response updateProject(Project t) {
		((DAO) hm.get(&quot;project&quot;)).update(t);
		return Response.status(Status.OK).entity(&quot;Project updated&quot;).build();
	}		
	@RolesAllowed(&quot;Administrator&quot;)
	@SuppressWarnings({ &quot;unchecked&quot;, &quot;rawtypes&quot; })
	@POST
	@Path(&quot;project&quot;)
	@Consumes(MediaType.APPLICATION_JSON)
	public Response persistProject(Project t) {
		((DAO) hm.get(&quot;project&quot;)).persist(t);
		return Response.status(Status.OK).entity(&quot;Project added&quot;).build();
	}
	
	/**
	 * Comment
	 */
	@RolesAllowed(&quot;Administrator&quot;)
	@SuppressWarnings({ &quot;unchecked&quot;, &quot;rawtypes&quot; })
	@DELETE
	@Path(&quot;comment/{id}&quot;)
	@Consumes(MediaType.APPLICATION_JSON)
	public Response deleteComment(@PathParam(&quot;id&quot;) int id) {
		((DAO) hm.get(&quot;comment&quot;)).delete(((DAO) hm.get(&quot;comment&quot;)).getById(id));
		return Response.status(Status.OK).entity(&quot;Comment deleted&quot;).build();
	}		
	@RolesAllowed(&quot;Administrator&quot;)
	@SuppressWarnings({ &quot;unchecked&quot;, &quot;rawtypes&quot; })
	@PUT
	@Path(&quot;comment&quot;)
	@Consumes(MediaType.APPLICATION_JSON)
	public Response updateComment(Comment t) {
		((DAO) hm.get(&quot;comment&quot;)).update(t);
		return Response.status(Status.OK).entity(&quot;Comment updated&quot;).build();
	}		
	@RolesAllowed(&quot;Administrator&quot;)
	@SuppressWarnings({ &quot;unchecked&quot;, &quot;rawtypes&quot; })
	@POST
	@Path(&quot;comment&quot;)
	@Consumes(MediaType.APPLICATION_JSON)
	public Response persistComment(Comment t) {
		((DAO) hm.get(&quot;comment&quot;)).persist(t);
		return Response.status(Status.OK).entity(&quot;Comment added&quot;).build();
	}
}
[/code]

Holy guacamoly, that's a brain f**ker. Let's do a deep dive.

* Within the constructor we register every model controller in a map where the key is the url path.
* `@RolesAllowed("Administrator")` is an annotation filtere by our Authenticator class. Only valid users with the administrator role will have access to this resource. I've left it
* `@PermitAll` is the opposite, no filtering here. I've only added this annotation to the `getAll` route so we have something to display in the browser later on.
* `@GET, @POST, @PUT, @DELETE` defines the method type of the route.
* `@Path("route")` defines the name of the url of access path.
* `@Consumes and @Produces` defines wether this action is consuming or returning data and also what kind of data.
* Have you seen the DAO casting? Cool isn't it? Thats why you should implement a class inteface whenever possible.

I hope you get the other lines of code. Lets move on to the authentication provider.

# Authentication provider

Our application uses basic http authentication header. That means we have to add a username and a password into the request header for every client access. With the authentication provider we check every request for a valid user and it's roles. The authentication provider implements the ContainerRequestFilter which allows you to filter specific client-server communication directions.

<img src="https://janikvonrotz.ch/wp-content/uploads/2015/03/Request-and-Response-Filter.png" alt="Request and Response Filter" width="936" height="206" class="aligncenter size-full wp-image-3275" />

ContainerRequestFilter - Filter/modify inbound requests
ContainerResponseFilter - Filter/modify outbound responses
ClientRequestFilter - Filter/modify outbound requests
ClientResponseFilter - Filter/modify inbound responses

* Update the authentication provier.

[code lang="java"]
package ch.issueman.webservice;

import java.io.IOException;
import java.lang.reflect.Method;
import java.util.Arrays;
import java.util.HashSet;
import java.util.List;
import java.util.Set;
import java.util.StringTokenizer;
import java.util.stream.Collectors;

import javax.annotation.security.DenyAll;
import javax.annotation.security.PermitAll;
import javax.annotation.security.RolesAllowed;
import javax.ws.rs.container.ContainerRequestContext;
import javax.ws.rs.container.ContainerRequestFilter;
import javax.ws.rs.core.MultivaluedMap;
import javax.ws.rs.core.Response;
import javax.ws.rs.core.Response.Status;
import javax.ws.rs.ext.Provider;

import org.jboss.resteasy.core.ResourceMethodInvoker;
import org.jboss.resteasy.util.Base64;

import ch.issueman.common.User;

@Provider
public class Authenticator implements ContainerRequestFilter {

	private static final String AUTHORIZATION_PROPERTY = &quot;Authorization&quot;;
	private static final String AUTHENTICATION_SCHEME = &quot;Basic&quot;;
	private static final Response ACCESS_DENIED =  Response.status(Status.UNAUTHORIZED).entity( &quot;Access denied for this resource.&quot; ).build( );
	private static final Response ACCESS_FORBIDDEN =  Response.status(Status.FORBIDDEN).entity( &quot;Nobody can access this resource.&quot; ).build( );
	
	@Override
	public void filter(ContainerRequestContext requestContext) {
		ResourceMethodInvoker methodInvoker = (ResourceMethodInvoker) requestContext.getProperty(&quot;org.jboss.resteasy.core.ResourceMethodInvoker&quot;);
        Method method = methodInvoker.getMethod();
		
		if (!method.isAnnotationPresent(PermitAll.class)) {
			
			if (method.isAnnotationPresent(DenyAll.class)) {
				requestContext.abortWith(ACCESS_FORBIDDEN);
				return;
			}

			final MultivaluedMap&lt;String, String&gt; headers = requestContext.getHeaders();
			final List&lt;String&gt; authorization = headers.get(AUTHORIZATION_PROPERTY);

			if (authorization == null || authorization.isEmpty()) {
				requestContext.abortWith(ACCESS_DENIED);
				return;
			}

			final String encodedUserPassword = authorization.get(0).replaceFirst(AUTHENTICATION_SCHEME + &quot; &quot;, &quot;&quot;);

			String usernameAndPassword = null;
			try {
				usernameAndPassword = new String(Base64.decode(encodedUserPassword));
			} catch (IOException e) {
				e.printStackTrace();
			}

			final StringTokenizer tokenizer = new StringTokenizer(usernameAndPassword, &quot;:&quot;);
			final String username = tokenizer.nextToken();
			final String password = tokenizer.nextToken();

			if (method.isAnnotationPresent(RolesAllowed.class)) {
				RolesAllowed rolesAnnotation = method.getAnnotation(RolesAllowed.class);
				Set&lt;String&gt; rolesSet = new HashSet&lt;String&gt;(Arrays.asList(rolesAnnotation.value()));

				if (!isUserAllowed(username, password, rolesSet)) {
					requestContext.abortWith(ACCESS_DENIED);
					return;
				}
			}
		}
	}

	private boolean isUserAllowed(final String username, final String password,	final Set&lt;String&gt; rolesSet) {
		boolean isAllowed = false;

		Controller&lt;User, Integer&gt; usercontroller = new Controller&lt;User, Integer&gt;(User.class);
		
		List&lt;User&gt; users = usercontroller.getAll().stream()
				.filter(p -&gt; p.getEmail().equals(username))
				.filter(p -&gt; p.getPassword().equals(password))
				.collect(Collectors.toList());
		
		if(users.get(0) != null){
			String userRole = users.get(0).getRole();
			
			if (rolesSet.contains(userRole)) {
				isAllowed = true;
			}
		}
		return isAllowed;
	}
}
[/code]

This class is an implementation of the `ContainerRequestFilter` interface. Wasn't sure wether I should explain this in detail. It's actually very simple: the request context contains our header data, from there it extracts, converts and validates the basic authentication data. Next the provider checks for added security annotations and runs related checks. With an additional function it looks up the user credentials and roles in our database via the user model controller.

This was the last piece of our controller. Finally we want to insert some data to dispaly and work with in the client.

# Insert fake data

The `javafaker` library creates random data on every method call. We will use this feature to fill our database.

* Update the seeder class.

**Seed.java**

[code lang="java"]
package ch.issueman.webservice;

import java.util.ArrayList;
import java.util.List;

import ch.issueman.common.Comment;
import ch.issueman.common.Employer;
import ch.issueman.common.Person;
import ch.issueman.common.Project;
import ch.issueman.common.User;

import com.github.javafaker.Faker;

public class Seed {

	public static void main(String[] args) {
		
		Faker faker = new Faker();

		Controller&lt;Person, Integer&gt; personcontroller = new Controller&lt;Person, Integer&gt;(Person.class);
		Controller&lt;User, Integer&gt; usercontroller = new Controller&lt;User, Integer&gt;(User.class);
		Controller&lt;Employer, Integer&gt; employercontroller = new Controller&lt;Employer, Integer&gt;(Employer.class);
		Controller&lt;Project, Integer&gt; projectcontroller = new Controller&lt;Project, Integer&gt;(Project.class);
		Controller&lt;Comment, Integer&gt; commentcontroller = new Controller&lt;Comment, Integer&gt;(Comment.class);

		usercontroller.deleteAll();
		employercontroller.deleteAll();
		projectcontroller.deleteAll();
		personcontroller.deleteAll();
		commentcontroller.deleteAll();
		
		int i = 0;
		int j = 0;
		
		for (i = 0; i &lt;= 20; i++) {

			Person person = new Person(faker.name().firstName());
			User user = new User(faker.name().firstName(), faker.internet().emailAddress(), faker.letterify(&quot;??????&quot;), &quot;Administrator&quot;);
			Employer employer = new Employer(faker.name().firstName(), faker.name().lastName());

			personcontroller.persist(person);
			usercontroller.persist(user);
			employercontroller.persist(employer);

			if (i % 4 == 0) {
				
				Project project = new Project(&quot;Project: &quot; + faker.name().lastName(), employer);
				
				List&lt;Comment&gt; comments = new ArrayList&lt;Comment&gt;();
				
				for (j = 0; j &lt;= 10; j++) {
					
					Comment comment = new Comment(faker.lorem().paragraph(),user);
					comments.add(comment);
					project.setComments(comments);
				}

				projectcontroller.persist(project);
			}
		}
		
		System.out.println(&quot;Seeded: &quot; + i*3 + &quot; people&quot;);
		System.out.println(&quot;Seeded: &quot; + (i/4+1) + &quot; projects&quot;);
		System.out.println(&quot;Seeded: &quot; + j*(i/4+1) + &quot; comments&quot;);
	}
}
[/code]

Do not run this class within eclipse, we will use gradle to execute the main method.
But before we do that we have to compile the classes and update the eclipse project metadata.

* Open your command line and navigate to the project directory: `cd <project>`
* Update the eclipse metadata: `gradle eclipse`
* Compile all projects: `gradle build`

Now we are ready to seed.

* Run the seed task that we configured in the first part of this tutorial: `gradle seed`

You should something like this:

[code]
Seeded: 63 people
Seeded: 6 projects
Seeded: 66 comments
[/code]

We got the code we got the data, it's time to run the webservice.

* On the command line start jetty with: `gradle jettyRun`
* Open your browser to: `http://localhost:8080/webservice/user`

If you see some json data, you did very well. If not, post the exception in the comment section and I'll help you.

In the next two part we are going to write a simple client application that consumes our webservice. See you.

# Update

* 2015-05-12 Added description for response and request filtering.

# Links

* [Part 1: Introduction and project setup](https://janikvonrotz.ch/2015/03/15/build-a-java-3-tier-application-from-scratch-part-1-introduction-and-project-setup/)
* [Part 2: Model setup](https://janikvonrotz.ch/2015/03/28/build-a-java-3-tier-application-from-scratch-part-2-model-setup/)
* [Part 3: Object-relational mapping](https://janikvonrotz.ch/2015/03/30/build-a-java-3-tier-application-from-scratch-part-3-object-relational-mapping/)
* [Part 4: Webservice](https://janikvonrotz.ch/2015/03/31/build-a-java-3-tier-application-from-scratch-part-4-webservice/)
* [Part 5: Client controller](https://janikvonrotz.ch/2015/04/01/build-a-java-3-tier-application-from-scratch-part-5-client-controller/)
* [Part 6: Client view](https://janikvonrotz.ch/2015/04/02/build-a-java-3-tier-application-from-scratch-part-6-client-view/)

# Source

[Securing JAX-RS RESTful services](http://www.java.cz/dwn/1003/72527_JAX_RS_Security_CZJUG_short.pdf)
[JAX-RS RESTEasy basic authentication and authorization tutorial](http://howtodoinjava.com/2013/07/25/jax-rs-2-0-resteasy-3-0-2-final-security-tutorial/)  
[Filtering JAX-RS Entities with Standard Security Annotations](http://blog.dejavu.sk/2014/02/04/filtering-jax-rs-entities-with-standard-security-annotations/)