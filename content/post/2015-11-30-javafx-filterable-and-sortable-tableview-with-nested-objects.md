---
id: 3286
title: 'JavaFX &#8211; Filterable and Sortable Tableview with nested Objects'
date: 2015-11-30T10:01:22+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=3286
permalink: /2015/11/30/javafx-filterable-and-sortable-tableview-with-nested-objects/
dsq_thread_id:
  - "4362336468"
image: /wp-content/uploads/2014/10/Java-logo.jpg
categories:
  - Java
tags:
  - cell
  - display
  - factory
  - filterable
  - form
  - java
  - javafx
  - nested
  - object
  - search
  - sortable
  - table
  - value
  - view
---
This is a simple example for a sortable and filterable table view with properties from nested objects.
<!--more-->
First step design a simple table view like this one:
<img src="https://janikvonrotz.ch/wp-content/uploads/2015/10/JavaFx-Filterable-Table-View.png" alt="JavaFx Filterable Table View" width="569" height="537" class="aligncenter size-full wp-image-3581" />

For this example I use a rather simple approach on the object relationship.

**Login** 1------1 **Person**

Every login is added to a person and vice versa. I won't go into detail, you have make assumptions or adapt other similar cases on this project.

In the person object we have some attributes that we want to display in the table view.

* Nachname
* Vorname
* Email

However the data access object we receive is the login. This is the challenge of nested objects and how to display these properties.

Now we will write the view class for our table view. The view class is connected with the javafx table view.
[code lang="java"]
public class PersonView{

	private static LoginData&lt;Login, Integer&gt; loginData= new LoginData&lt;Login, Integer&gt;(Login.class);
	private FilteredList&lt;Login&gt; filteredData = new FilteredList&lt;Login&gt;(FXCollections.observableArrayList(),	p -&gt; true);

	@FXML
	private TableView&lt;Login&gt; tvData;

	@FXML
	private TextField txFilter;

	@FXML
	private TableColumn&lt;Login, String&gt; tcNachname;

	@FXML
	private TableColumn&lt;Login, String&gt; tcVorname;

	@FXML
	private TableColumn&lt;Login, String&gt; tcEmail;
[/code]
The LoginData component of course holds the login objects and communicates with the persistence layer.
The filtered list will be synced with the table view component.
As next, we tell what properties should be displayed in the cells. Remember that we want to display nested properties.

[code lang="java"]
	@Override
	public void initialize(URL arg0, ResourceBundle arg1) {
		
		tcNachname.setCellValueFactory(new Callback&lt;TableColumn.CellDataFeatures&lt;Login,String&gt;,ObservableValue&lt;String&gt;&gt;() {  
			public ObservableValue&lt;String&gt; call(CellDataFeatures&lt;Login, String&gt; param) {
				return new SimpleStringProperty(param.getValue().getPerson().getNachname());
			}  
		});
		
		tcVorname.setCellValueFactory(new Callback&lt;TableColumn.CellDataFeatures&lt;Login,String&gt;,ObservableValue&lt;String&gt;&gt;() {  
			public ObservableValue&lt;String&gt; call(CellDataFeatures&lt;Login, String&gt; param) {
				return new SimpleStringProperty(param.getValue().getPerson().getVorname());
			}  
		});
		tcEmail.setCellValueFactory(new Callback&lt;TableColumn.CellDataFeatures&lt;Login,String&gt;,ObservableValue&lt;String&gt;&gt;() {  
			public ObservableValue&lt;String&gt; call(CellDataFeatures&lt;Login, String&gt; param) {
				return new SimpleStringProperty(param.getValue().getPerson().getEmail());
			}  
		});

[/code]

By adding a cell value factory you can overwrite the logic of how values of the list should displayed and access.
Now it's time for the filter. By adding a listener to the text property we hook in code that will trigger whenever something is entered into the filter text field.

[code lang="java"]

		txFilter.textProperty().addListener(
				(observable, oldValue, newValue) -&gt; {
					filteredData.setPredicate(t -&gt; {
						
							if (newValue == null || newValue.isEmpty()) {
								return true;
							}

							String lowerCaseFilter = newValue.toLowerCase();
							String objectvalues = t.getPerson().getNachname() 
									+ t.getPerson().getVorname()
									+ t.getPerson().getEmail();
											
							if (objectvalues.toLowerCase().indexOf(lowerCaseFilter) != -1) {
								return true; 
							}

							return false;
						});
				});		

		Refresh();
	}

[/code]

Finally create a refresh method that loads the data into the table view. Make sure to make this method public.

[code lang="java"]

	public void Refresh() {
		try {	
			List&lt;Login&gt; list = loginData.getAll();
			filteredData = new FilteredList&lt;Login&gt;(FXCollections.observableArrayList(list),	p -&gt; true);
			SortedList&lt;Login&gt; sortedData = new SortedList&lt;Login&gt;(filteredData);
			sortedData.comparatorProperty().bind(tvData.comparatorProperty());
			tvData.setItems(sortedData);
		} catch (Exception e) {
			SomeDialogLauncher.showError(e);
		}
	}
}
[/code]

Congrats, that's it.