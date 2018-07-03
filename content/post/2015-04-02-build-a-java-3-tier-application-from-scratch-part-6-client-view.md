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

```xml
<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.scene.image.*?>
<?import javafx.scene.effect.*?>
<?import javafx.scene.paint.*?>
<?import javafx.scene.control.*?>
<?import javafx.scene.text.*?>
<?import java.lang.*?>
<?import javafx.scene.layout.*?>
<?import javafx.scene.layout.AnchorPane?>

<AnchorPane maxHeight="-Infinity" maxWidth="-Infinity" minHeight="-Infinity" minWidth="-Infinity" prefHeight="342.0" prefWidth="528.0" xmlns="http://javafx.com/javafx/8" xmlns:fx="http://javafx.com/fxml/1" fx:controller="ch.issueman.client.HomeView">
   <effect>
      <Glow />
   </effect>
   <children>
      <Pane fx:id="pnData" prefHeight="342.0" prefWidth="528.0" visible="false">
         <children>
            <TableView fx:id="tvEmployer" onMouseClicked="#clickTableView" prefHeight="342.0" prefWidth="317.0">
               <columns>
                  <TableColumn fx:id="tcId" prefWidth="50.0" text="ID" />
                  <TableColumn fx:id="tcName" prefWidth="135.0" text="Name" />
                  <TableColumn fx:id="tcCompany" prefWidth="131.0" text="Company" />
               </columns>
            </TableView>
            <TextField fx:id="txName" layoutX="327.0" layoutY="83.0" />
            <TextField fx:id="txCompany" layoutX="327.0" layoutY="127.0" />
            <Button fx:id="btAdd" layoutX="327.0" layoutY="171.0" mnemonicParsing="false" onAction="#clickAdd" text="Add" />
            <Button fx:id="btUpdate" layoutX="374.0" layoutY="171.0" mnemonicParsing="false" onAction="#clickUpdate" text="Update" />
            <Button fx:id="btDelete" layoutX="443.0" layoutY="171.0" mnemonicParsing="false" onAction="#clickDelete" text="Delete" />
         </children>
      </Pane>
      <Pane fx:id="pnLogin" prefHeight="342.0" prefWidth="528.0">
         <children>
            <TextField fx:id="txUsername" layoutX="171.0" layoutY="104.0" />
            <PasswordField fx:id="pfPassword" layoutX="171.0" layoutY="140.0" />
            <Button fx:id="btLogin" layoutX="236.0" layoutY="190.0" mnemonicParsing="false" onAction="#clickLogin" text="Login" />
         </children>
      </Pane>
   </children>
</AnchorPane>
```

Every view requires a controller, see there `fx:controller="ch.issueman.client.HomeView"`. I've called this controller `<view name>View.java` in order to avoid using the word controller too much.

* Upate the view handler.

**HomeView.java**

```java
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

	private static Controller<Employer, Integer> controller = new Controller<Employer, Integer>(Employer.class, null);

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
	private TableView<Employer> tvEmployer;

	@FXML
	private TableColumn<Employer, Integer> tcId;

	@FXML
	private TableColumn<Employer, String> tcName;
	
	@FXML
	private TableColumn<Employer, String> tcCompany;
	
	public void initialize(URL arg0, ResourceBundle arg1) {
		tcId.setCellValueFactory(new PropertyValueFactory<Employer, Integer>("id"));
		tcName.setCellValueFactory(new PropertyValueFactory<Employer, String>("name"));
		tcCompany.setCellValueFactory(new PropertyValueFactory<Employer, String>("company"));
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
		User user = new User("", txUsername.getText(), pfPassword.getText(), "");
		controller = new Controller<Employer, Integer>(Employer.class, user);
		if(controller.login()){
			pnData.setVisible(true);
			pnLogin.setVisible(false);
		}else{
			txUsername.setText("");
			pfPassword.setText("");
		}
	}
}
```

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