---
id: 3071
title: Superb Java development with Gradle, Hibernate, FlywayDB, JavaFX and Eclipse – Part 2
date: 2015-03-05T17:01:57+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=3071
permalink: /2015/03/05/superb-java-development-with-gradle-hibernate-flywaydb-javafx-and-eclipse-part-2/
dsq_thread_id:
  - "3569968870"
image: /wp-content/uploads/2014/10/Java-logo.jpg
categories:
  - Java
tags:
  - application
  - builder
  - button
  - call
  - click
  - column
  - event
  - fxml
  - hibernate
  - interface
  - java
  - javafx
  - methods
  - rich
  - scene
  - table
  - tutorial
  - windows
---
As promised here's the sequel to my last tutorial: <a href="https://janikvonrotz.ch/2015/03/03/superb-java-development-with-gradle-hibernate-flywaydb-javafx-and-eclipse-part-1/" title="Superb Java development with Gradle, Hibernate, FlywayDB, JavaFX and Eclipse – Part 1">Part 1</a>

This time we create a basic rich application with JavaFX. It will be quite simple, but still a good example to show how you can structure a complex project according to MVC.
<!--more-->
# Update

We will start with a refresh of the file structure. The picture below shows yellow marked the new packages and directories. On the right side is a preview of the application window labeled with the type of each component.

<img src="https://janikvonrotz.ch/wp-content/uploads/2015/03/Updated-IssueMan-Filestructure.png" alt="Updated IssueMan Filestructure" width="1000" height="450" class="aligncenter size-full wp-image-3073" />

* Create the new packages.
* Move the files to their new folders. Eclipse will automatically update the namespace and import commands.
* In the hibernate configuration file update the mapping property.

**hibernate.cfg.xml**

[code lang="xml"]
&lt;mapping class=&quot;ch.hslu.issueman.model.Person&quot;/&gt;
[/code]

* Finally create the JavaFX view file `Home.fxml` in the view directory. We will update this file later.

# Install

To create the visual layout I've used [JavaFX Scene Builder](http://www.oracle.com/technetwork/java/javase/downloads/javafxscenebuilder-info-2157684.html).
Please install this tool, we will need to edit the `Home.fxml` file.

# Coding - Controller

To connect the view with the Person Controller we have to provide scene builder objects and methods.
I assume you've completed the first tutorial and will only show the changes to file.

**PersonController.java** - begin

[code lang="java"]
import java.net.URL;
import java.util.List;
import java.util.ResourceBundle;
import javafx.collections.FXCollections;
import javafx.fxml.FXML;
import javafx.fxml.Initializable;
import javafx.scene.control.Button;
import javafx.scene.control.TableColumn;
import javafx.scene.control.TableView;
import javafx.scene.control.TextField;
import javafx.scene.control.cell.PropertyValueFactory;
import ch.hslu.issueman.model.Person;
[/code]

A lot of new dependencies required by JavaFX.

At the end of the file before the closing `}` inser the following snippets.

[code lang="java"]
	@FXML
	private Button btAdd;
	
	@FXML
	private Button btUpdate;
	
	@FXML
	private Button btDelete;
	
	@FXML
	private TextField txName;
	
	@FXML
	private TableView&lt;Person&gt; tvPerson;
	
	@FXML
	private TableColumn&lt;Person, Integer&gt; tcId;
	
	@FXML
	private TableColumn&lt;Person, String&gt; tcName;
[/code]

These objects allows us to set and get values from the application interface.

[code lang="java"]
	@Override
	public void initialize(URL arg0, ResourceBundle arg1) {
		tcId.setCellValueFactory(new PropertyValueFactory&lt;Person, Integer&gt;(&quot;id&quot;));		
		tcName.setCellValueFactory(new PropertyValueFactory&lt;Person, String&gt;(&quot;name&quot;));
		refreshPersonTable();
	}
[/code]

Whenever the application is startet the `initialize` method will be called. In this function we tell the table view column which value of the person it has to display and also call the `refreshPersonTable` method which is added next.

[code lang="java"]
	public void refreshPersonTable(){
		tvPerson.setItems(FXCollections.observableArrayList(getAll()));
	}
[/code]

In this method we feed the table view with a person array.

[code lang="java"]	
	@FXML
	public void clickTableView(){
		txName.textProperty().set(tvPerson.getSelectionModel().getSelectedItem().getName());
	}
[/code]

As you can see there's a FXML-Annotation as well. This tag allows us to connect an event added by the scene build with a function in the controller.
Whenever a row is clicked in the table view we load the content to the form part (if you have got more than one field its recommanded to store the chosen object in a temporary variable).

[code lang="java"]
	@FXML
	public void clickAdd(){
		controller.persist(new Person(txName.getText()));
		refreshPersonTable();
	}
	
	@FXML
	public void clickUpdate(){
		Person p = tvPerson.getSelectionModel().getSelectedItem();
		p.setName(txName.getText());
		controller.update(p);
		refreshPersonTable();
	}
	
	@FXML
	public void clickDelete(){
		controller.delete(tvPerson.getSelectionModel().getSelectedItem());
		refreshPersonTable();
	}
[/code]

Some more event methods for the buttons.

# Design

Designing the user interface is very easy.

<img src="https://janikvonrotz.ch/wp-content/uploads/2015/03/Scene-Builder-Issue-Manager.png" alt="Scene Builder Issue Manager" width="1069" height="490" class="aligncenter size-full wp-image-3084" />

* Open the `Home.fxml` file with scene builder.
* Connect the scene with the PersonController.
* Add the components showed in the preview and connect them with the fxml objects and methods.

If you don't know how to accomplish that yet you can copy the fxml definitions below.

**Home.fxml**

[code lang="xml"]
&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot;?&gt;

&lt;?import javafx.scene.image.*?&gt;
&lt;?import javafx.scene.effect.*?&gt;
&lt;?import javafx.scene.paint.*?&gt;
&lt;?import javafx.scene.control.*?&gt;
&lt;?import javafx.scene.text.*?&gt;
&lt;?import java.lang.*?&gt;
&lt;?import javafx.scene.layout.*?&gt;
&lt;?import javafx.scene.layout.AnchorPane?&gt;

&lt;AnchorPane maxHeight=&quot;-Infinity&quot; maxWidth=&quot;-Infinity&quot; minHeight=&quot;-Infinity&quot; minWidth=&quot;-Infinity&quot; prefHeight=&quot;300.0&quot; prefWidth=&quot;400.0&quot; xmlns=&quot;http://javafx.com/javafx/8&quot; xmlns:fx=&quot;http://javafx.com/fxml/1&quot; fx:controller=&quot;ch.hslu.issueman.controller.PersonController&quot;&gt;
   &lt;effect&gt;
      &lt;Glow /&gt;
   &lt;/effect&gt;
   &lt;children&gt;
      &lt;TableView fx:id=&quot;tvPerson&quot; onMouseClicked=&quot;#clickTableView&quot; prefHeight=&quot;307.0&quot; prefWidth=&quot;187.0&quot;&gt;
         &lt;columns&gt;
            &lt;TableColumn fx:id=&quot;tcId&quot; prefWidth=&quot;50.0&quot; text=&quot;ID&quot; /&gt;
            &lt;TableColumn fx:id=&quot;tcName&quot; prefWidth=&quot;135.0&quot; text=&quot;Name&quot; /&gt;
         &lt;/columns&gt;
      &lt;/TableView&gt;
      &lt;Button fx:id=&quot;btAdd&quot; layoutX=&quot;200.0&quot; layoutY=&quot;99.0&quot; mnemonicParsing=&quot;false&quot; onMouseClicked=&quot;#clickAdd&quot; text=&quot;Add&quot; /&gt;
      &lt;Button fx:id=&quot;btUpdate&quot; layoutX=&quot;248.0&quot; layoutY=&quot;99.0&quot; mnemonicParsing=&quot;false&quot; onMouseClicked=&quot;#clickUpdate&quot; text=&quot;Update&quot; /&gt;
      &lt;Button fx:id=&quot;btDelete&quot; layoutX=&quot;317.0&quot; layoutY=&quot;99.0&quot; mnemonicParsing=&quot;false&quot; onMouseClicked=&quot;#clickDelete&quot; text=&quot;Delete&quot; /&gt;
      &lt;TextField fx:id=&quot;txName&quot; layoutX=&quot;196.0&quot; layoutY=&quot;61.0&quot; /&gt;
   &lt;/children&gt;
&lt;/AnchorPane&gt;
[/code]

And we are almost done.

# Coding - Application

To launch the whole scene we have to update the `App.java` code as showed below.

**App.java**

[code lang="java"]
package ch.hslu.issueman;
	
import javafx.application.Application;
import javafx.fxml.FXMLLoader;
import javafx.stage.Stage;
import javafx.scene.Scene;

public class App extends Application {
	@Override
	public void start(Stage primaryStage) {
		try {
			primaryStage.setResizable(false);
			primaryStage.setTitle(&quot;Issue Manager&quot;);
			primaryStage.setScene(new Scene(FXMLLoader.load(getClass().getResource(&quot;view/Home.fxml&quot;))));
			primaryStage.show();			
		} catch(Exception e) {
			e.printStackTrace();
		}
	}
	
	public static void main(String[] args) {
		launch(args);
	}
}

[/code]

Hurray! We are done! You can run the application now and should be able to add, edit and delete people from the list all whilst they are stored in the MySQL database.

# Source

You can download the source code of the project here: [https://github.com/janikvonrotz/issue-manager/releases/tag/1.1](https://github.com/janikvonrotz/issue-manager/releases/tag/1.1)
