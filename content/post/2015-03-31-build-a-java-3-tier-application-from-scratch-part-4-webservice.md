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

```xml
<web-app id="WebApp_ID" version="2.4"
	xmlns="http://java.sun.com/xml/ns/j2ee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee 
	http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
	<display-name>Restful Web Application</display-name>

	<context-param>
		<param-name>resteasy.resources</param-name>
		<param-value>ch.issueman.webservice.Route</param-value>
	</context-param>

	<context-param>
		<param-name>resteasy.providers</param-name>
		<param-value>ch.issueman.webservice.Authenticator</param-value>
	</context-param>

	<servlet>
		<servlet-name>resteasy-servlet</servlet-name>
		<servlet-class>
			org.jboss.resteasy.plugins.server.servlet.HttpServletDispatcher
		</servlet-class>
	</servlet>

	<servlet-mapping>
		<servlet-name>resteasy-servlet</servlet-name>
		<url-pattern>/*</url-pattern>
	</servlet-mapping>

</web-app>
```

The resources class mapps actions to url paths, a provider hooks up into every request done by a client such as authentication or access filtering.

Our Route class file has the most lines of code. But don't be anxious, it contains a lot of repetetive configurations. For every model we have to define the CRUD methods. A DRY (don't repeat yourself) approach is very difficult here, as a lot is going on behind the scenes. I only could stack the `get` methods together.
That means 5 models * 3 CUD-methods + 2 R-methods + 1 login method = 18 path actions for our webservice.

* Update the Route class.

**Route.java**

```java
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

@Path("/")
public class Route{
	
	private Map <String, DAO<?, Integer>> hm = new HashMap<String, DAO<?, Integer>>();

	public Route(){
		hm.put("person", new Controller<Person, Integer>(Person.class));
		hm.put("user", new Controller<User, Integer>(User.class));
		hm.put("employer", new Controller<Employer, Integer>(Employer.class));
		hm.put("project", new Controller<Project, Integer>(Project.class));
		hm.put("comment", new Controller<Comment, Integer>(Comment.class));	
	}
	
	@RolesAllowed("Administrator")
	@GET
	@Path("{entity}/{id}")
	@Produces(MediaType.APPLICATION_JSON)
	public Model getEntityById(@PathParam("entity") String entity, @PathParam("id") int id) {
		return (Model) hm.get(entity).getById(id);
	} 	
	
	@PermitAll
	@SuppressWarnings("unchecked")
	@GET
	@Path("{entity}")
	@Produces(MediaType.APPLICATION_JSON)
	public List<Model> getAll(@PathParam("entity") String entity) {
		return (List<Model>) hm.get(entity).getAll();
	}	
	
	@PermitAll
	@SuppressWarnings({ "unchecked" })
	@POST
	@Path("login")
	@Consumes(MediaType.APPLICATION_JSON)
	@Produces(MediaType.APPLICATION_JSON)
	public Response login(User user) {
		
		List<User> users = ((List<User>) hm.get("user").getAll()).stream()
				.filter(p -> p.getEmail().equals(user.getEmail()))
				.filter(p -> p.getPassword().equals(user.getPassword()))
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
	@RolesAllowed("Administrator")
	@SuppressWarnings({ "unchecked", "rawtypes" })
	@DELETE
	@Path("person/{id}")
	@Consumes(MediaType.APPLICATION_JSON)
	public Response deletePerson(@PathParam("id") int id) {
		((DAO) hm.get("person")).delete(((DAO) hm.get("person")).getById(id));
		return Response.status(Status.OK).entity("Person deleted").build();
	}		
	@RolesAllowed("Administrator")
	@SuppressWarnings({ "unchecked", "rawtypes" })
	@PUT
	@Path("person")
	@Consumes(MediaType.APPLICATION_JSON)
	public Response updatePerson(Person t) {
		((DAO) hm.get("person")).update(t);
		return Response.status(Status.OK).entity("Person updated").build();
	}		
	@RolesAllowed("Administrator")
	@SuppressWarnings({ "unchecked", "rawtypes" })
	@POST
	@Path("person")
	@Consumes(MediaType.APPLICATION_JSON)
	public Response persistPerson(Person t) {
		((DAO) hm.get("person")).persist(t);
		return Response.status(Status.OK).entity("Person added").build();
	}
		
	/**
	 * Employer
	 */
	@RolesAllowed("Administrator")
	@SuppressWarnings({ "unchecked", "rawtypes" })
	@DELETE
	@Path("employer/{id}")
	@Consumes(MediaType.APPLICATION_JSON)
	public Response deleteEmployer(@PathParam("id") int id) {
		((DAO) hm.get("employer")).delete(((DAO) hm.get("employer")).getById(id));
		return Response.status(Status.OK).entity("Employer deleted").build();
	}		
	@RolesAllowed("Administrator")
	@SuppressWarnings({ "unchecked", "rawtypes" })
	@PUT
	@Path("employer")
	@Consumes(MediaType.APPLICATION_JSON)
	public Response updateEmployer(Employer t) {
		((DAO) hm.get("employer")).update(t);
		return Response.status(Status.OK).entity("Employer updated").build();
	}		
	@RolesAllowed("Administrator")
	@SuppressWarnings({ "unchecked", "rawtypes" })
	@POST
	@Path("employer")
	@Consumes(MediaType.APPLICATION_JSON)
	public Response persistEmployer(Employer t) {
		((DAO) hm.get("employer")).persist(t);
		return Response.status(Status.OK).entity("Employer added").build();
	}
	
	/**
	 * User
	 */
	@RolesAllowed("Administrator")
	@SuppressWarnings({ "unchecked", "rawtypes" })
	@DELETE
	@Path("user/{id}")
	@Consumes(MediaType.APPLICATION_JSON)
	public Response deleteUser(@PathParam("id") int id) {
		((DAO) hm.get("user")).delete(((DAO) hm.get("user")).getById(id));
		return Response.status(Status.OK).entity("User deleted").build();
	}		
	@RolesAllowed("Administrator")
	@SuppressWarnings({ "unchecked", "rawtypes" })
	@PUT
	@Path("user")
	@Consumes(MediaType.APPLICATION_JSON)
	public Response updateUser(User t) {
		((DAO) hm.get("user")).update(t);
		return Response.status(Status.OK).entity("User updated").build();
	}		
	@RolesAllowed("Administrator")
	@SuppressWarnings({ "unchecked", "rawtypes" })
	@POST
	@Path("user")
	@Consumes(MediaType.APPLICATION_JSON)
	public Response persistUser(User t) {
		((DAO) hm.get("user")).persist(t);
		return Response.status(Status.OK).entity("User added").build();
	}
	
	/**
	 * Project
	 */
	@RolesAllowed("Administrator")
	@SuppressWarnings({ "unchecked", "rawtypes" })
	@DELETE
	@Path("project/{id}")
	@Consumes(MediaType.APPLICATION_JSON)
	public Response deleteProject(@PathParam("id") int id) {
		((DAO) hm.get("project")).delete(((DAO) hm.get("project")).getById(id));
		return Response.status(Status.OK).entity("Project deleted").build();
	}		
	@RolesAllowed("Administrator")
	@SuppressWarnings({ "unchecked", "rawtypes" })
	@PUT
	@Path("project")
	@Consumes(MediaType.APPLICATION_JSON)
	public Response updateProject(Project t) {
		((DAO) hm.get("project")).update(t);
		return Response.status(Status.OK).entity("Project updated").build();
	}		
	@RolesAllowed("Administrator")
	@SuppressWarnings({ "unchecked", "rawtypes" })
	@POST
	@Path("project")
	@Consumes(MediaType.APPLICATION_JSON)
	public Response persistProject(Project t) {
		((DAO) hm.get("project")).persist(t);
		return Response.status(Status.OK).entity("Project added").build();
	}
	
	/**
	 * Comment
	 */
	@RolesAllowed("Administrator")
	@SuppressWarnings({ "unchecked", "rawtypes" })
	@DELETE
	@Path("comment/{id}")
	@Consumes(MediaType.APPLICATION_JSON)
	public Response deleteComment(@PathParam("id") int id) {
		((DAO) hm.get("comment")).delete(((DAO) hm.get("comment")).getById(id));
		return Response.status(Status.OK).entity("Comment deleted").build();
	}		
	@RolesAllowed("Administrator")
	@SuppressWarnings({ "unchecked", "rawtypes" })
	@PUT
	@Path("comment")
	@Consumes(MediaType.APPLICATION_JSON)
	public Response updateComment(Comment t) {
		((DAO) hm.get("comment")).update(t);
		return Response.status(Status.OK).entity("Comment updated").build();
	}		
	@RolesAllowed("Administrator")
	@SuppressWarnings({ "unchecked", "rawtypes" })
	@POST
	@Path("comment")
	@Consumes(MediaType.APPLICATION_JSON)
	public Response persistComment(Comment t) {
		((DAO) hm.get("comment")).persist(t);
		return Response.status(Status.OK).entity("Comment added").build();
	}
}
```

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

```java
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

	private static final String AUTHORIZATION_PROPERTY = "Authorization";
	private static final String AUTHENTICATION_SCHEME = "Basic";
	private static final Response ACCESS_DENIED =  Response.status(Status.UNAUTHORIZED).entity( "Access denied for this resource." ).build( );
	private static final Response ACCESS_FORBIDDEN =  Response.status(Status.FORBIDDEN).entity( "Nobody can access this resource." ).build( );
	
	@Override
	public void filter(ContainerRequestContext requestContext) {
		ResourceMethodInvoker methodInvoker = (ResourceMethodInvoker) requestContext.getProperty("org.jboss.resteasy.core.ResourceMethodInvoker");
        Method method = methodInvoker.getMethod();
		
		if (!method.isAnnotationPresent(PermitAll.class)) {
			
			if (method.isAnnotationPresent(DenyAll.class)) {
				requestContext.abortWith(ACCESS_FORBIDDEN);
				return;
			}

			final MultivaluedMap<String, String> headers = requestContext.getHeaders();
			final List<String> authorization = headers.get(AUTHORIZATION_PROPERTY);

			if (authorization == null || authorization.isEmpty()) {
				requestContext.abortWith(ACCESS_DENIED);
				return;
			}

			final String encodedUserPassword = authorization.get(0).replaceFirst(AUTHENTICATION_SCHEME + " ", "");

			String usernameAndPassword = null;
			try {
				usernameAndPassword = new String(Base64.decode(encodedUserPassword));
			} catch (IOException e) {
				e.printStackTrace();
			}

			final StringTokenizer tokenizer = new StringTokenizer(usernameAndPassword, ":");
			final String username = tokenizer.nextToken();
			final String password = tokenizer.nextToken();

			if (method.isAnnotationPresent(RolesAllowed.class)) {
				RolesAllowed rolesAnnotation = method.getAnnotation(RolesAllowed.class);
				Set<String> rolesSet = new HashSet<String>(Arrays.asList(rolesAnnotation.value()));

				if (!isUserAllowed(username, password, rolesSet)) {
					requestContext.abortWith(ACCESS_DENIED);
					return;
				}
			}
		}
	}

	private boolean isUserAllowed(final String username, final String password,	final Set<String> rolesSet) {
		boolean isAllowed = false;

		Controller<User, Integer> usercontroller = new Controller<User, Integer>(User.class);
		
		List<User> users = usercontroller.getAll().stream()
				.filter(p -> p.getEmail().equals(username))
				.filter(p -> p.getPassword().equals(password))
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
```

This class is an implementation of the `ContainerRequestFilter` interface. Wasn't sure wether I should explain this in detail. It's actually very simple: the request context contains our header data, from there it extracts, converts and validates the basic authentication data. Next the provider checks for added security annotations and runs related checks. With an additional function it looks up the user credentials and roles in our database via the user model controller.

This was the last piece of our controller. Finally we want to insert some data to dispaly and work with in the client.

# Insert fake data

The `javafaker` library creates random data on every method call. We will use this feature to fill our database.

* Update the seeder class.

**Seed.java**

```java
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

		Controller<Person, Integer> personcontroller = new Controller<Person, Integer>(Person.class);
		Controller<User, Integer> usercontroller = new Controller<User, Integer>(User.class);
		Controller<Employer, Integer> employercontroller = new Controller<Employer, Integer>(Employer.class);
		Controller<Project, Integer> projectcontroller = new Controller<Project, Integer>(Project.class);
		Controller<Comment, Integer> commentcontroller = new Controller<Comment, Integer>(Comment.class);

		usercontroller.deleteAll();
		employercontroller.deleteAll();
		projectcontroller.deleteAll();
		personcontroller.deleteAll();
		commentcontroller.deleteAll();
		
		int i = 0;
		int j = 0;
		
		for (i = 0; i <= 20; i++) {

			Person person = new Person(faker.name().firstName());
			User user = new User(faker.name().firstName(), faker.internet().emailAddress(), faker.letterify("??????"), "Administrator");
			Employer employer = new Employer(faker.name().firstName(), faker.name().lastName());

			personcontroller.persist(person);
			usercontroller.persist(user);
			employercontroller.persist(employer);

			if (i % 4 == 0) {
				
				Project project = new Project("Project: " + faker.name().lastName(), employer);
				
				List<Comment> comments = new ArrayList<Comment>();
				
				for (j = 0; j <= 10; j++) {
					
					Comment comment = new Comment(faker.lorem().paragraph(),user);
					comments.add(comment);
					project.setComments(comments);
				}

				projectcontroller.persist(project);
			}
		}
		
		System.out.println("Seeded: " + i*3 + " people");
		System.out.println("Seeded: " + (i/4+1) + " projects");
		System.out.println("Seeded: " + j*(i/4+1) + " comments");
	}
}
```

Do not run this class within eclipse, we will use gradle to execute the main method.
But before we do that we have to compile the classes and update the eclipse project metadata.

* Open your command line and navigate to the project directory: `cd <project>`
* Update the eclipse metadata: `gradle eclipse`
* Compile all projects: `gradle build`

Now we are ready to seed.

* Run the seed task that we configured in the first part of this tutorial: `gradle seed`

You should something like this:

```
Seeded: 63 people
Seeded: 6 projects
Seeded: 66 comments
```

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