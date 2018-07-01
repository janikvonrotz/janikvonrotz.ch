---
id: 3183
title: 'Build a Java 3-tier application from scratch – Part 6: Client view'
date: 2015-04-02T07:13:31+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=3183
permalink: /2015/04/02/build-a-java-3-tier-application-from-scratch-part-6-client-view/
dsq_thread_id:
  - "3648008847"
image: /wp-content/uploads/2014/10/Java-logo.jpg
categories:
  - Java
tags:
  - 3-tier
  - application
  - client
  - controller
  - graphical
  - interface
  - java
  - javafx
  - user
  - view
---
Welcome to the final part of my tutorial. In this last part we are going to write our view and run the client application.

The client application consists of a login and a data pane. When you've successfully logged in, the visibility of these panes will be switched. That's all you have to know. In case you want to use scene builder to create the client GUI here's a picture of what you have to build:
<!--more-->
<img src="https://janikvonrotz.ch/wp-content/uploads/2015/03/client-Scene-Builder-1024x305.png" alt="client Scene Builder" width="720" height="214" class="aligncenter size-large wp-image-3189" />

# View

If you're too lazy simply copy the fxml definitions.

* Update the Home fxml markup file.

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

&lt;AnchorPane maxHeight=&quot;-Infinity&quot; maxWidth=&quot;-Infinity&quot; minHeight=&quot;-Infinity&quot; minWidth=&quot;-Infinity&quot; prefHeight=&quot;342.0&quot; prefWidth=&quot;528.0&quot; xmlns=&quot;http://javafx.com/javafx/8&quot; xmlns:fx=&quot;http://javafx.com/fxml/1&quot; fx:controller=&quot;ch.issueman.client.HomeView&quot;&gt;
   &lt;effect&gt;
      &lt;Glow /&gt;
   &lt;/effect&gt;
   &lt;children&gt;
      &lt;Pane fx:id=&quot;pnData&quot; prefHeight=&quot;342.0&quot; prefWidth=&quot;528.0&quot; visible=&quot;false&quot;&gt;
         &lt;children&gt;
            &lt;TableView fx:id=&quot;tvEmployer&quot; onMouseClicked=&quot;#clickTableView&quot; prefHeight=&quot;342.0&quot; prefWidth=&quot;317.0&quot;&gt;
               &lt;columns&gt;
                  &lt;TableColumn fx:id=&quot;tcId&quot; prefWidth=&quot;50.0&quot; text=&quot;ID&quot; /&gt;
                  &lt;TableColumn fx:id=&quot;tcName&quot; prefWidth=&quot;135.0&quot; text=&quot;Name&quot; /&gt;
                  &lt;TableColumn fx:id=&quot;tcCompany&quot; prefWidth=&quot;131.0&quot; text=&quot;Company&quot; /&gt;
               &lt;/columns&gt;
            &lt;/TableView&gt;
            &lt;TextField fx:id=&quot;txName&quot; layoutX=&quot;327.0&quot; layoutY=&quot;83.0&quot; /&gt;
            &lt;TextField fx:id=&quot;txCompany&quot; layoutX=&quot;327.0&quot; layoutY=&quot;127.0&quot; /&gt;
            &lt;Button fx:id=&quot;btAdd&quot; layoutX=&quot;327.0&quot; layoutY=&quot;171.0&quot; mnemonicParsing=&quot;false&quot; onAction=&quot;#clickAdd&quot; text=&quot;Add&quot; /&gt;
            &lt;Button fx:id=&quot;btUpdate&quot; layoutX=&quot;374.0&quot; layoutY=&quot;171.0&quot; mnemonicParsing=&quot;false&quot; onAction=&quot;#clickUpdate&quot; text=&quot;Update&quot; /&gt;
            &lt;Button fx:id=&quot;btDelete&quot; layoutX=&quot;443.0&quot; layoutY=&quot;171.0&quot; mnemonicParsing=&quot;false&quot; onAction=&quot;#clickDelete&quot; text=&quot;Delete&quot; /&gt;
         &lt;/children&gt;
      &lt;/Pane&gt;
      &lt;Pane fx:id=&quot;pnLogin&quot; prefHeight=&quot;342.0&quot; prefWidth=&quot;528.0&quot;&gt;
         &lt;children&gt;
            &lt;TextField fx:id=&quot;txUsername&quot; layoutX=&quot;171.0&quot; layoutY=&quot;104.0&quot; /&gt;
            &lt;PasswordField fx:id=&quot;pfPassword&quot; layoutX=&quot;171.0&quot; layoutY=&quot;140.0&quot; /&gt;
            &lt;Button fx:id=&quot;btLogin&quot; layoutX=&quot;236.0&quot; layoutY=&quot;190.0&quot; mnemonicParsing=&quot;false&quot; onAction=&quot;#clickLogin&quot; text=&quot;Login&quot; /&gt;
         &lt;/children&gt;
      &lt;/Pane&gt;
   &lt;/children&gt;
&lt;/AnchorPane&gt;
[/code]

Every view requires a controller, see there `fx:controller="ch.issueman.client.HomeView"`. I've called this controller `<view name>View.java` in order to avoid using the word controller too much.

* Upate the view handler.

**HomeView.java**

[code lang="java"]
package ch.issueman.client;

import java.net.URL;
import java.util.ResourceBundle;

import javafx.collections.FXCollections;
import javafx.fxml.FXML;
import javafx.fxml.Initializable;
import javafx.scene.control.Button;
import javafx.scene.control.PasswordField;
import javafx.scene.control.TableColumn;
import javafx.scene.control.TableView;
import javafx.scene.control.TextField;
import javafx.scene.control.cell.PropertyValueFactory;
import javafx.scene.layout.Pane;
import ch.issueman.common.Employer;
import ch.issueman.common.User;

public class HomeView implements Initializable {

	private static Controller&lt;Employer, Integer&gt; controller = new Controller&lt;Employer, Integer&gt;(Employer.class, null);

	@FXML
	private Button btAdd;

	@FXML
	private Button btUpdate;

	@FXML
	private Button btDelete;

	@FXML
	private Button btLogin;
	
	@FXML
	private Pane pnLogin;
	
	@FXML
	private Pane pnData;
	
	@FXML
	private TextField txUsername;
	
	@FXML
	private TextField txName;
	
	@FXML
	private TextField txCompany;
	
	@FXML
	private PasswordField pfPassword;
	
	@FXML
	private TableView&lt;Employer&gt; tvEmployer;

	@FXML
	private TableColumn&lt;Employer, Integer&gt; tcId;

	@FXML
	private TableColumn&lt;Employer, String&gt; tcName;
	
	@FXML
	private TableColumn&lt;Employer, String&gt; tcCompany;
	
	public void initialize(URL arg0, ResourceBundle arg1) {
		tcId.setCellValueFactory(new PropertyValueFactory&lt;Employer, Integer&gt;(&quot;id&quot;));
		tcName.setCellValueFactory(new PropertyValueFactory&lt;Employer, String&gt;(&quot;name&quot;));
		tcCompany.setCellValueFactory(new PropertyValueFactory&lt;Employer, String&gt;(&quot;company&quot;));
		refreshPersonTable();
	}

	public void refreshPersonTable() {
		tvEmployer.setItems(FXCollections.observableArrayList(controller.getAll()));
	}

	@FXML
	public void clickTableView() {
		Employer e = tvEmployer.getSelectionModel().getSelectedItem();
		txName.textProperty().set(e.getName());
		txCompany.textProperty().set(e.getCompany());
	}

	@FXML
	public void clickAdd() {
		controller.persist(new Employer(txName.getText(), txCompany.getText()));
		refreshPersonTable();
	}

	@FXML
	public void clickUpdate() {
		Employer e = tvEmployer.getSelectionModel().getSelectedItem();
		e.setName(txName.getText());
		e.setCompany(txCompany.getText());
		controller.update(e);
		refreshPersonTable();
	}

	@FXML
	public void clickDelete() {
		Employer e = tvEmployer.getSelectionModel().getSelectedItem();
		controller.delete(e);
		refreshPersonTable();
	}
	
	@FXML
	public void clickLogin(){
		User user = new User(&quot;&quot;, txUsername.getText(), pfPassword.getText(), &quot;&quot;);
		controller = new Controller&lt;Employer, Integer&gt;(Employer.class, user);
		if(controller.login()){
			pnData.setVisible(true);
			pnLogin.setVisible(false);
		}else{
			txUsername.setText(&quot;&quot;);
			pfPassword.setText(&quot;&quot;);
		}
	}
}
[/code]

Make sure that you understand this class, it's not that difficult. Every GUI components is connected to variables and every action (click, focus, etc.) requires a method in the view handler.

Finally open the command line.

* Navigate to the project directory: `cd <project>`
* Start the webservice: `gradle jettyRun`
* Run the `App.java` file within eclipse.
* Open the browser on `http://localhost:8080/webservice/user` copy the login data from one of the users and login in with your client.

If you did everything right you should see the employer entities.

Congratulations :D in case you got this far you are a professional Java developer from now on.

Thanks for joining and a nice feedback in comments section.

# Download Code

You can download the project from here: [https://github.com/janikvonrotz/issue-manager/releases/tag/v2.2](https://github.com/janikvonrotz/issue-manager/releases/tag/v2.2)

# Links

* [Part 1: Introduction and project setup](https://janikvonrotz.ch/2015/03/15/build-a-java-3-tier-application-from-scratch-part-1-introduction-and-project-setup/)
* [Part 2: Model setup](https://janikvonrotz.ch/2015/03/28/build-a-java-3-tier-application-from-scratch-part-2-model-setup/)
* [Part 3: Object-relational mapping](https://janikvonrotz.ch/2015/03/30/build-a-java-3-tier-application-from-scratch-part-3-object-relational-mapping/)
* [Part 4: Webservice](https://janikvonrotz.ch/2015/03/31/build-a-java-3-tier-application-from-scratch-part-4-webservice/)
* [Part 5: Client controller](https://janikvonrotz.ch/2015/04/01/build-a-java-3-tier-application-from-scratch-part-5-client-controller/)
* [Part 6: Client view](https://janikvonrotz.ch/2015/04/02/build-a-java-3-tier-application-from-scratch-part-6-client-view/)