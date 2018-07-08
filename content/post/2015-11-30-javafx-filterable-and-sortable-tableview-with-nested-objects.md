---
title: 'JavaFX - Filterable and Sortable Tableview with nested Objects'
date: 2015-11-30T10:01:22+00:00
author: Janik von Rotz
slug: javafx-filterable-and-sortable-tableview-with-nested-objects
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
![JavaFx Filterable Table View](/wp-content/uploads/2015/10/JavaFx-Filterable-Table-View.png)

For this example I use a rather simple approach on the object relationship.

**Login** 1------1 **Person**

Every login is added to a person and vice versa. I won't go into detail, you have make assumptions or adapt other similar cases on this project.

In the person object we have some attributes that we want to display in the table view.

* Nachname
* Vorname
* Email

However the data access object we receive is the login. This is the challenge of nested objects and how to display these properties.

Now we will write the view class for our table view. The view class is connected with the javafx table view.
```java
public class PersonView{

	private static LoginData<Login, Integer> loginData= new LoginData<Login, Integer>(Login.class);
	private FilteredList<Login> filteredData = new FilteredList<Login>(FXCollections.observableArrayList(),	p -> true);

	@FXML
	private TableView<Login> tvData;

	@FXML
	private TextField txFilter;

	@FXML
	private TableColumn<Login, String> tcNachname;

	@FXML
	private TableColumn<Login, String> tcVorname;

	@FXML
	private TableColumn<Login, String> tcEmail;
```
The LoginData component of course holds the login objects and communicates with the persistence layer.
The filtered list will be synced with the table view component.
As next, we tell what properties should be displayed in the cells. Remember that we want to display nested properties.

```java
	@Override
	public void initialize(URL arg0, ResourceBundle arg1) {
		
		tcNachname.setCellValueFactory(new Callback<TableColumn.CellDataFeatures<Login,String>,ObservableValue<String>>() {  
			public ObservableValue<String> call(CellDataFeatures<Login, String> param) {
				return new SimpleStringProperty(param.getValue().getPerson().getNachname());
			}  
		});
		
		tcVorname.setCellValueFactory(new Callback<TableColumn.CellDataFeatures<Login,String>,ObservableValue<String>>() {  
			public ObservableValue<String> call(CellDataFeatures<Login, String> param) {
				return new SimpleStringProperty(param.getValue().getPerson().getVorname());
			}  
		});
		tcEmail.setCellValueFactory(new Callback<TableColumn.CellDataFeatures<Login,String>,ObservableValue<String>>() {  
			public ObservableValue<String> call(CellDataFeatures<Login, String> param) {
				return new SimpleStringProperty(param.getValue().getPerson().getEmail());
			}  
		});

```

By adding a cell value factory you can overwrite the logic of how values of the list should displayed and access.
Now it's time for the filter. By adding a listener to the text property we hook in code that will trigger whenever something is entered into the filter text field.

```java

		txFilter.textProperty().addListener(
				(observable, oldValue, newValue) -> {
					filteredData.setPredicate(t -> {
						
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

```

Finally create a refresh method that loads the data into the table view. Make sure to make this method public.

```java

	public void Refresh() {
		try {	
			List<Login> list = loginData.getAll();
			filteredData = new FilteredList<Login>(FXCollections.observableArrayList(list),	p -> true);
			SortedList<Login> sortedData = new SortedList<Login>(filteredData);
			sortedData.comparatorProperty().bind(tvData.comparatorProperty());
			tvData.setItems(sortedData);
		} catch (Exception e) {
			SomeDialogLauncher.showError(e);
		}
	}
}
```

Congrats, that's it.