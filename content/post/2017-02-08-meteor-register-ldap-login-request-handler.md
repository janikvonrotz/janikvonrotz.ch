---
id: 4195
title: Meteor register LDAP login request handler
date: 2017-02-08T20:53:56+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4195
permalink: /2017/02/08/meteor-register-ldap-login-request-handler/
dsq_thread_id:
  - "5534292034"
image: /wp-content/uploads/2017/02/Meteor-Logo-inverted.png
categories:
  - Meteor
tags:
  - active
  - auth
  - authentication
  - custom
  - directory
  - handler
  - ldap
  - login
  - meteor
  - request
---
One requirement for my current Meteor project was that a user must login with their ActiveDirectory account. This means that Meteor must be able to authenticate against LDAP. In atmosphere there are already a few packages available which implement and support LDAP authentication. However, they are either outdated or difficult to configure. This is why I've decided to build my own custom login request handler for Meteor. 
<!--more-->
In the next steps I will explain, how you can implement and register your own custom login handler and authenticate user with their corporate LDAP account. I assume you already know how to configure and use the `accounts-base` and `accounts-password` Meteor packages.

Let's get startet with the client.

**client/users/actions.js**

[code lang="js"]
...
let loginUserWithLDAP = (email, password, callback) =&gt; {
    var loginRequest = {
      ldap: true,
      email: email,
      pass: password,
    }
    Accounts.callLoginMethod({
      methodArguments: [loginRequest],
      userCallback: callback
    })
  }

  loginUserWithLDAP(email, password, (error, result) =&gt; {
    if (!error) {
    ...
[/code] 

Instead of using `Meteor.loginWithPassword()` you have to do a login method call with a different set of parameters. This allows us to recognise from the server if client intends to authenticate with LDAP. Make sure that you don't pass a `password` option as part of the login request, otherwise the `accounts-password` login handler will throw an error.

Next I will tell you how to create and register the LDAP authentication handler in three parts.

**server/ldap.js**

[code lang="js"]
import ldap from 'ldapjs'
import assert from 'assert'
import { Accounts } from 'meteor/accounts-base'
import Future from 'fibers/future'

var ldapAuth = {
  url: 'ldap://ldap.forumsys.com',
  searchOu: 'dc=example,dc=com',
  searchQuery: (email) =&gt; {
    return {
      filter: `(mail=${email})`,
      scope: 'sub'
    }
  }
}

ldapAuth.checkAccount = (options) =&gt; {
  options = options || {}

  ldapAuth.client = ldap.createClient({
    url: ldapAuth.url
  })

  let dn = []
  var future = new Future()
  ...
[/code]

After the library imports, options for the LDAP authentication are defined. Instead of connecting the LDAP client to a real LDAP directory I've used the public available directory of Forum Systems: [Online LDAP Test Server](http://www.forumsys.com/tutorials/integration-how-to/ldap/online-ldap-test-server/). Following up the function header of the authentication method and a suspicious object is declared. You might haven't seen or read about the Future fiber yet. As you might know Meteor doesn't like async code the same as Node does or you came along a situation where your asynchronous code didn't work as expected. To keep it short, the authentication handler request code must be run synchronous and the Future fiber helps us running asynchronous code.

[code lang="js"]
  ...
  ldapAuth.client.search(ldapAuth.searchOu, ldapAuth.searchQuery(options.email), (error, result) =&gt; {
    assert.ifError(error)

    result.on('searchEntry', (entry) =&gt; {
      dn.push(entry.objectName)
      return ldapAuth.profile = {
        firstname: entry.object.cn,
        lastname: entry.object.sn
      }
    })

    result.on('error', function(error){
      throw new Meteor.Error(500, &quot;LDAP server error&quot;)
    })

    return result.on('end', function(){

      if (dn.length === 0) {
        future['return'](false)
        return false
      }

      return ldapAuth.client.bind(dn[0], options.pass, (error) =&gt; {

        if (error) {
          future['return'](false)
          return false
        }

        return ldapAuth.client.unbind((error) =&gt; {
          assert.ifError(error)
          return future['return'](!error)
        })
      })  
    })
  })
  return future.wait()
}
...
[/code]

Now comes probably the most difficult part. The body of our auth method tells if the LDAP credentials are valid by binding and unbinding the LDAP user with the LDAP directory. Any misbehaviour results in the return value `false`. An important line to point out here is the return statement of the `ldapAuth` object which is also assigned with a new `profile` property. In case of successful authentication we will use this property to create a new Meteor user in the users collection in the next step.

[code lang="js"]
...
Accounts.registerLoginHandler('ldap', (loginRequest) =&gt; {

  if (!loginRequest.ldap) {
    return undefined
  }

  if (ldapAuth.checkAccount(loginRequest)) {
    var userId = null
    var user = Meteor.users.findOne({ &quot;emails.address&quot; : loginRequest.email })
    if (!user) {
      userId = Accounts.createUser({
        email: loginRequest.email,
        password: loginRequest.pass,
        profile: ldapAuth.profile,
        roles: ['user'],
      })
      Meteor.users.update(userId, { $set: { 'emails.0.verified': true } })
    } else {
      userId = user._id
    }

    let stampedToken = Accounts._generateStampedLoginToken()
    let hashStampedToken = Accounts._hashStampedToken(stampedToken)
    Meteor.users.update(userId,
      { $push: { 'services.resume.loginTokens': hashStampedToken } }
    )

    return {
      userId: userId,
      token: stampedToken.token
    }
  }
})
...
[/code]

Finally, in case of successful LDAP user check, a collection lookup finds out wether the authenticated users is already in the database and if not creates a new entry. As you can see the `profile` property of the `ldapAuth` object is now used as a parameter. To make sure that user is authenticated after a browser refresh you have to create a token and store it. The return object contains the user identity and the login token.

Of course you can adapt this example and use it to authenticate against another provider. Other accounts packages such as `accounts-facebook` and the `accounts-password` work almost the same way.