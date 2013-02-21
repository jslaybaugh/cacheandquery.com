---
layout: post
title: Customizing ASP.NET MVC Basic Authentication
tags: 
- asp.net mvc
- security
- technology
status: publish
type: post
published: true
---

## Introduction

About a week ago, I set out to add authentication to an API for a small project I was working on. I didn't need the complexity of something like [OAuth][1] and for an API, Forms Authentication doesn't make much sense. I figured the easiest would be to just enable Basic Authentication in IIS and I'd be on my way. Well, the first problem with using Basic Authentication as it comes in IIS is that it only connects to Windows accounts, which in my case wouldn't work; I needed to authenticate against a database. So, I started Googling... without much immediate success. My two most successful results were [here][2] and [here][3]. The first one wasn't bad, but it also wasn't complete enough for me to wrap my head around it and make it work. The second, by [Alex James][4], is awesome, but it's written more in the context of WCF and therefore uses HTTP Modules and I also wasn't too fond of the way he suggested handling allowing unauthenticated access to some endpoints. I really liked the simple beauty of the [Authorize] attribute in MVC and wanted to go more in that direction. So, I combined the two and with a little bit of my own special sauce, came up with what you'll see here. It allows for flexibility in use (selectively tagging actions and/or controllers) and also in structure (dependency injection using Ninject). An added bonus I discovered is that it doesn't even depend on IIS since all we are really doing is returning a standard 401 HTTP error code and adding another header to the response stream.

## Getting Started

First thing we need to do is make sure all authentication (besides anonymous) is turned off in whatever server you are using, and that our app is configured to do no authentication of its own (in web.config).

<script src="https://gist.github.com/jslaybaugh/5002173.js?file=web.config"></script>

![Anonymous authentication enabled in IIS 7](/img/posts/iis-auth.png)

Now that all other authentication is turned off, we'll create a custom attribute to do the our custom authentication against our custom data store.

## Making It Work

This is the full source for the class that defines the CustomBasicAuthorize attribute that we'll use to decorate our controllers and actions that we want to secure.

<script src="https://gist.github.com/jslaybaugh/5002173.js?file=CustomBasicAuthorizeAttribute.cs"></script>    
I also added a new subclass called HttpCustomBasicUnauthorizedResult. This just kept the code cleaner, IMO, but the real important part is making sure that the "WWW-Authenticate" header gets passed back with the response. This, in combination with the 401 HTTP Error that is assigned by the base class, is what makes the client know that basic authentication is being required.

<script src="https://gist.github.com/jslaybaugh/5002173.js?file=HttpCustomBasicUnauthorizedResult.cs"></script>

## Putting It To Use

For this sample app, we'll create a simple AccountController with two methods. One, called Authenticate, will not be secured (so we can get authenticated) and the other, called Me, will require authentication and display information about the logged in user.

<script src="https://gist.github.com/jslaybaugh/5002173.js?file=AccountController.cs"></script>

And in this case, the repository (as seen below) is just validating against or supplying some hard coded values. In this case, the username/password combination of "user" and "pass" will get you logged in and then once you are logged in, it will display those same hard coded values as JSON.


<script src="https://gist.github.com/jslaybaugh/5002173.js?file=HardCodedUserRepository.cs"></script>

![Authenticated JSON Output](/img/posts/auth-json-output.png)

The nice thing about this model is that since it uses an IPrincipal, all the parts of authentication you are probably already used to still work. You can use User.IsInRole() for manual testing of privileges or you can add properties to your attribute to only allow certain users or roles to specific actions or controllers. For example, an attribute like:

`[CustomBasicAuthorize(Users="user, otheruser", Roles="admin, manager")]`

will allow only the specified users and roles to access whatever action/controller it is decorating.

## Wrapping Up

So, with only two classes (not counting any of the repository or dependency injection code), we've now created a very extensible way to easily secure any/all of our API methods so that any client can now pass credentials using a standardized protocol!

One thing that I hope goes without saying is that if you are using Basic Authentication for anything, it should **always** be done over SSL. Even though the credentials are Base64 encoded, they are still just plain text and can be easily decoded and viewed by anyone inspecting the transmission (it only took us one line of code to decode it above).

The full source code for this project is available on [GitHub][5].

Happy Coding!

 [1]: http://oauth.net/
 [2]: http://stackoverflow.com/questions/1407153/rest-ful-basic-authentication-with-asp-net-mvc
 [3]: http://blogs.msdn.com/b/astoriateam/archive/2010/07/21/odata-and-authentication-part-6-custom-basic-authentication.aspx
 [4]: http://twitter.com/adjames
 [5]: https://github.com/jslaybaugh/MvcCustomizedBasicAuthentication  

