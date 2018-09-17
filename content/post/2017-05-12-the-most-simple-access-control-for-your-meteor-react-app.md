---
title: The most simple access control for your Meteor React app
date: 2017-05-12T09:46:48+00:00
author: Janik von Rotz
slug: the-most-simple-access-control-for-your-meteor-react-app
images:
  - /wp-content/uploads/2017/05/RBAC-Meteor.png
categories:
  - Blog
  - Meteor
  - React
tags:
  - access
  - acl
  - action
  - control
  - easy
  - example
  - list
  - meteor
  - permission
  - react
  - role
  - simple
  - user
---
For my last Meteor React app I've designed the most simple role based access control. The basic idea is that users can have multiple roles and every action possible is only allowed by a specified set of roles. For my Meteor React app the following scenarios were considered: 
<!--more-->

Only users with specific roles are allowed to...

 * call a Meteor method.
 * subscribe to a publication.
 * display React components in the UI.

To solve this problem a role based access control (RBAC) has been implemented:

user -> roles -> action -> permission for publication / method / component

Now you might miss the url access control as a scenario, why did I ignore it? Well, the answer is simple, a user won't access what he can't see. As long as we hide buttons and links which might lead to a restricted view and control access on the data layer properly, there's no need to have access control for urls.

**/imports/helpers/config.js**

The config file merges the access control list (acl) and Meteor settings. For every possible action a set of roles is defined.

```
import { Meteor } from 'meteor/meteor'
import { objectAssign } from './index'

let acl = {
  // routers permissions
  'routers.read': [ 'admin', 'spec', 'tech', 'user' ],
  'routers.insert': [ 'admin', 'spec', 'tech' ],
  'routers.update': [ 'admin', 'spec', 'tech' ],
  'routers.remove': [ 'admin', 'spec' ],
  'routers.restore': [ 'admin' , 'spec' ],
  'routers.export': [ 'admin', 'spec' ],

  // notification permissions
  'notifications.read': [ 'admin', 'spec', 'tech' ],
  'notifications.receive': [ 'admin', 'spec', 'tech' ],
  'notifications.insert': [ 'admin', 'spec', 'tech' ],
  'notifications.remove': [ 'admin', 'spec', 'tech' ],
  'notifications.export': [ 'admin', 'spec' ],
}

export default objectAssign(Meteor.settings.private, Meteor.settings.public, { acl: acl })
```

Of course you could create a collection to store the acl and make the permission model dynamic.

**/imports/helpers/isAllowed.js**

This is the only function required to check the users permission.

```
import { config } from './index'

export default (action, roles) => {
  let allowed = false
  let allowedRoles = config.acl[action]
  roles = roles != null ? roles : []
  roles.map((role) => {
     allowed = allowedRoles.indexOf(role) != -1
  })
  return allowed
}
```

Simple isn't it?

**/server/methods/routers.js**

This example shows how the permission is checked in a Meteor method.

```
...
import { isAllowed } from '/imports/helpers'

export default () => {
  Meteor.methods({
    'routers.insert'(object) {
      check(object, Object)

      // check permissions
      let roles = Meteor.userId() ? Meteor.user().roles : null
      if (!isAllowed('routers.insert', roles)) {
        throw new Meteor.Error(i18n.error.insufficent_rights, i18n.message.insufficent_rights_for_method)
      }

      // insert object
      ...
```

Make sure the permission check is the first thing done when the method is executed.

**/server/publication/router.js**

Now we restrict access on the data access layer.

```
...
import { isAllowed } from '/imports/helpers'

export default () => {

  Meteor.publish('routers.list', function(selector = {}) {

    // check permissions
    let user = Meteor.users.findOne(this.userId)
    let roles = user ? user.roles : null
    if (isAllowed('routers.read', roles)) {
      return Routers.find(selector)
    } else {
      this.stop()
      return
    }
  })
  ...
```

The user roles are accessible using `this.user` in a publication.

**client/routers/Router.js**

Finally you can restrict the visibility of React components quite easily.

```
...
import { isAllowed } from '/imports/helpers'

class Router extends React.Component {
  ...
  render() {
    let { loading, user, i18n } = this.props
    return loading ? <CircularProgress /> : <Card>
          ...

          { isAllowed('routers.update', user ? user.roles : null) ?
          <RaisedButton
          type="submit"
          label={ i18n.button.update }
          primary={ true } />
          : null }

          { isAllowed('routers.remove', user ? user.roles : null) ?
          <RaisedButton
          onTouchTap={ this.toggleDialog.bind(this, 'openRemoveDialog') }
          label={ i18n.button.remove }
          secondary={ true } />
          : null }

          ...
    </Card>
  }
}

export default Router
```

As you can see the `isAllowed` function is used for all scenarios.

[![Untitled](/wp-content/uploads/2017/05/Meteor-React-component-access-control.gif)](/wp-content/uploads/2017/05/Meteor-React-component-access-control.gif)

Do you like this solution? Leave a comment and tell my more.

