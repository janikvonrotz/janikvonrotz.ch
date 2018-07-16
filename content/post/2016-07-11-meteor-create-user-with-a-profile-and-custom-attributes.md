---
title: Meteor create user with a profile and custom attributes
date: 2016-07-11T20:34:48+00:00
author: Janik von Rotz
slug: meteor-create-user-with-a-profile-and-custom-attributes
dsq_thread_id:
  - "4977952562"
image: /wp-content/uploads/2015/12/Meteor-Logo.png
categories:
  - Meteor
tags:
  - account
  - admin
  - attribute
  - create
  - custom
  - editable
  - email
  - field
  - firstname
  - lastname
  - meteor
  - password
  - profile
  - protected
  - react
  - specific
  - user
  - username
---
For Meteor there are not many options left when choosing a user account package. The built-in option is the only use- and successful solution so far. The package is well documented and works like a charm. However, whenever I set up the account system in Meteor I am confronted with these two scenarios: 

* How can I extend the user profile object with data from a registration form? (A user should be able to edit the profile data himself later on.)
* And how can I add other attributes to the user collection object? (A user should not be able to change a custom attribute himself later on.)

These are two fundamental obstacles almost every developer faces when setting up the account system. There are a lot of solutions out there on how to do this in Meteor properly, but a lot of them are poorly described and make it difficult the get the right idea of how the account system works. It got even more difficult due  to API changes and Meteor itself that changed a lot over years. Now I would like to give a good example for this two questions. 
<!--more-->
The example below is an excerpt from one of my Meteor Mantra apps.

Lets get started. On the server-side I'll give you this simple boilerplate:

**server\configs\account.js**

```
import {Meteor} from 'meteor/meteor';
import { Accounts } from 'meteor/accounts-base';

export default () => {

  Accounts.onCreateUser((options, user) => {

    user.profile = options.profile ? options.profile : {};
    user.admin = options.admin;

    // send verification mail
    // Meteor.setTimeout(function() {
    //   Accounts.sendVerificationEmail(user._id);
    // }, 2 * 1000);

    return user;
  });
}
```

It's basically is a hook for the client create method which make sure the user object properties are at the right place. Make sure to remember what is passed by the options variable and what not, I'll promise it's not what you'll expect from the client side.

For the client side I'll show you a method which is called inside a React component.

**client\modules\user\component\users.jsx**

```

...

  insert() {
    var user = {
      email: this.refs.email.getValue(),
      password: this.refs.password.getValue(),
      profile:{
       firstname: this.refs.firstname.getValue(),
       lastname: this.refs.lastname.getValue(),
     },
     admin: this.refs.admin.isChecked(),
    }
    this.props.insert(user)
  }

...

```

The object created in this method I pass to the `Accounts.createUser(user, [err, res])` method. As you can see there aren't any options on the client side method, so how does Meteor distinct the properties into user and option variables for the server? Well everything that is passed with the user object and is not called **email**, **username** and **password** is put into the options variable. 

That was it, there's not much more. Basically you can adapt the user creation process from Meteor to any of your requirements now.