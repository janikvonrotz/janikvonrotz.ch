---
id: 638
title: ADFS Login Customization
date: 2013-10-18T13:04:41+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=638
permalink: /2013/10/18/adfs-login-customization/
dsq_thread_id:
  - "1871576330"
image: /wp-content/uploads/2013/09/office-logo_v3.jpg
categories:
  - Office 365
tags:
  - adfs
  - custom
  - edit
  - hide
  - hint
  - login
  - modifications
  - office365
  - page
  - simple
  - title
---
The purpose of this article is to show the intention and implemention of the most common modifications for the ADFS login page.

Out of the box the login form should look like this:

[![ADFS Login page](/wp-content/uploads/2013/10/ADFS-Login-page.png)](/wp-content/uploads/2013/10/ADFS-Login-page.png)The f.e. is by default the page url, this can confuse the user, due they expect something like "your login" or "Office365 Login".

<!--more-->

<h1>logo</h1>

To add a logo you simply edit the <strong>web.config</strong> file.

<ul>
    <li>Comment out the section:</li>
</ul>

```

<!--
<add key=”logo” value=”logo.png” />
-->

```

<ul>
    <li>And add the path to your logo file.</li>
</ul>

<h1>“hint” label [domainusername]</h1>

This label is part of the page's localization, that means you have to edit the language resource file.

The localization files are stored under <strong>App_GlobalResources</strong> there you'll find one file for every language CommonResources.<strong>en</strong>.resx.

Edit the file of your language an replace the "hint" with you definition.

<h1>Remove the heading title</h1>

In the root folder of the login page in the folder  <strong>MasterPages</strong> is a file named <strong>MasterPage.master</strong>. This file defines the look of the login page. To hide the title just comment this part out.

```
<!--
<div class="GroupLargeMargin">
<div class="TextSizeXLarge">
<asp:Label ID="STSLabel" runat"server"></asp:Label>
</div>
</div>
-->
```

<h1>Auto-Populate the Username Field of the Forms Sign-in Page When Signing in to Office 365</h1>

With Office365 and ADFS it's possible to login from the <a href="https://login.microsoftonline.com" target="_blank">microsoft Office365 login page</a> or via a<a href="https://community.office365.com/en-us/wikis/sso/using-smart-links-or-idp-initiated-authentication-with-office-365.aspx" target="_blank"> smart link</a> to get directly redirected to the ADFS login page.

The Office365 login page will process the username you enter and if it's a on premise user you'll get redirected to the ADFS login page. By default the ADFS login page doesn't care about the username you've provided at the Office365 login page. To populate the username into the login field the ADFS page has be modified like this:

<h2>Modify global.asax.cs</h2>

Process username from the url parameter.

<ul>
    <li>Open global.asax.cs for editing</li>
    <li>Find the following and set your cursor to the next line down:</li>
</ul>

```

public void Application_BeginRequest()
{

```

<ul>
    <li>Paste the following code:</li>
</ul>

```

HttpRequest request = HttpContext.Current.Request;
HttpResponse response = HttpContext.Current.Response;

if ( !String.IsNullOrEmpty( request.Params["username"] ) )
{
     HttpCookie cookie = new HttpCookie( "Office365Username", request.Params["username"] );
     cookie.Expires = DateTime.UtcNow.AddMinutes( 10 );
     Response.Cookies.Add( cookie );
}

```

<ul>
    <li>Save and Close global.asax.cs</li>
</ul>

<h2>Modify FormsSignIn.aspx.cs</h2>

Add the username parameter to the username box.

<ul>
    <li>Open FormsSignIn.aspx.cs for editing</li>
    <li>Find the following and set your cursor to the next line down:</li>
</ul>

```

using System;

```

Paste the following code:

```

using System.Web;

```

Find the following and set your cursor to the next line down:

```

protected void Page_Load( object sender, EventArgs e )
{

```

Paste the following code:

```

HttpCookie cookie = Context.Request.Cookies.Get( "Office365Username" );

if ( null != cookie &amp;&amp; !String.IsNullOrEmpty( cookie.Value ) )
{
     UsernameTextBox.Text = cookie.Value;
     cookie.Expires = DateTime.UtcNow.AddDays( -1 );
     cookie.Value = "";
     Context.Response.Cookies.Add( cookie );
}

```

Save and Close FormsSignIn.aspx.cs