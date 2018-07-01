---
id: 3921
title: 'Meteor and Mantra - Routing redirect'
date: 2016-05-26T10:03:22+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=3921
permalink: /2016/05/26/meteor-and-mantra-routing-redirect/
dsq_thread_id:
  - "4858703770"
image: /wp-content/uploads/2016/05/Mantra-Logo.jpeg
categories:
  - Meteor
tags:
  - architecture
  - config
  - group
  - logic
  - mantra
  - meteor
  - redirect
  - rights
  - role
  - route
  - routing
  - solution
  - trigger
  - user
---
Recently I started using [Mantra]() to develop my Meteor apps. As with any other framework You'll find a lot of answers to common questions in a related forum or any other resource. However I couldn't get a good answer on how to implement routing logic properly. The Mantra specs also don't tell much about how to approach it. So here's my solution.
<!--more-->
First I want to tell you about a little drawback, which is essential to understand the whole issue with routing in Mantra. The Flowrouter redirect statement can only work in the `componentDidMount` directive of an React component. 

```js
componentDidMount(){
  FlowRouter.go('/login');
}
```

As I don't want load a component and then doing a redirect I had to make the redirect on a higher level of the Mantra architecture.

So where else could it be than in the route file?
To give you a better understanding I'll use the post module which is part of a blogging app.

**client/modules/post/routes.jsx**

```js
...

  FlowRouter.route('/posts', {
    name: 'post.list',
    triggersEnter: [function(context, redirect) {
      actions.posts.access_route('post.list', redirect);
    }],
    action() {
      mount(AppLayout, {
        content: () => (<MainPage />)
      });
    }
  });

...
```

As you can see I'll use a trigger action, despite the Mantra specs tell me not do so. For now it's the only practical approach.

Import the action in the route and call the `access_route` function.

**client/modules/post/actions/posts.js**

```js
import {cannot_access, redirect_login, redirect_verify} from '/lib/access_control';

...

access_route(routename, redirect) {
    if(redirect_login(routename)){
      redirect('/login');
    } else if(redirect_verify()){
      redirect('/email-verification');
    } else if(cannot_access(routename)){
      redirect('/');
    }
  }

...
```

This is done for every module. Then you get a custom routing logic for every module while still using the same function to verify f.g. a user can access a specific route.

**/lib/access_control.js**

```js
// client side
export function redirect_login(routename){
  var roles = _.findWhere(Meteor.settings.public.routes, {name: routename}).roles;
  if(!_.contains(roles, "Public") &amp;&amp; !Meteor.userId()){
    console.log("redirect login");
    return true;
  }
  return false;
};

// client side
export function redirect_verify(){
  if(Meteor.userId() &amp;&amp; !Meteor.user().emails[0].verified){
    console.log("redirect verify");
    return true;
  }
  return false;
};

// client side
export function cannot_access(routename){
  var roles = _.findWhere(Meteor.settings.public.routes, {name: routename}).roles;
  if(_.contains(roles, "Public") || Roles.userIsInRole(Meteor.user(), roles)){
    console.log("allow route " + routename);
    return false;
  }
  console.log("deny route " + routename);
  return true;
};
```

Here I'll bundle everything that is related in someway with permissions and access rights. The functions in this library are either used by the client or the server.  I will use alanning roles to check role membership later on.

When assigning and checking for permissions with user, groups and rules I always use this schema:

**Object <-> Access Group <-> Role <-> User**

This schema is not a new invention, it's a copy of Microsoft's AGDLP idea.

Finally update the settings.json with access rules. Make sure to add it to the public part, otherwise the client can't load the settings.

**settings.json**

```
...

{
  "name": "post.list",
  "roles": [
     "Admin",
     "Author",
     "Manager"
  ]
},

...

```

Yay! you did it. So do you like my idea? Would like to hear from you in the comment section.