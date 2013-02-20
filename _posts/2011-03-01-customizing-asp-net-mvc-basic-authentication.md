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

``

[![IIS 7 Authentication][6]][6]
Anonymous authentication enabled in IIS 7

Now that all other authentication is turned off, we'll create a custom attribute to do the our custom authentication against our custom data store.

## Making It Work

This is the full source for the class that defines the CustomBasicAuthorize attribute that we'll use to decorate our controllers and actions that we want to secure.

    public class CustomBasicAuthorizeAttribute: AuthorizeAttribute
    	{
    		bool _RequireSsl = true;
    		public bool RequireSsl
    		{
    			get { return _RequireSsl; }
    			set { _RequireSsl = value; }
    		}
    
    		// These lines of code are only required when using Ninject for
    		// constructor dependency injection with a parameterless constructor
    		// of this attribute.
    		[Inject]
    		public IUserRepository Repository { get; set; }
    		public CustomBasicAuthorizeAttribute()
    		{
    			// NinjectHelper is a static helper class that just exposes the current kernel object
    			// check the code in NinjectHttpApplicationModule.cs to see where the kernel is initialized
    			NinjectHelper.Kernel.Inject(this);
    		}
    		// end DI code
    
    		private void CacheValidateHandler(HttpContext context, object data, ref HttpValidationStatus validationStatus)
    		{
    			validationStatus = OnCacheAuthorization(new HttpContextWrapper(context));
    		}
    
    		public override void OnAuthorization(AuthorizationContext filterContext)
    		{
    			if (filterContext == null) throw new ArgumentNullException("filterContext");
    
    			if (!Authenticate(filterContext.HttpContext))
    			{
    				// HttpCustomBasicUnauthorizedResult inherits from HttpUnauthorizedResult and does the
    				// work of displaying the basic authentication prompt to the client
    				filterContext.Result = new HttpCustomBasicUnauthorizedResult();
    			}
    			else
    			{
    				// AuthorizeCore is in the base class and does the work of checking if we have
    				// specified users or roles when we use our attribute
    				if (AuthorizeCore(filterContext.HttpContext))
    				{
    					HttpCachePolicyBase cachePolicy = filterContext.HttpContext.Response.Cache;
    					cachePolicy.SetProxyMaxAge(new TimeSpan(0));
    					cachePolicy.AddValidationCallback(CacheValidateHandler, null /* data */);
    				}
    				else
    				{
    					// auth failed, display login
    
    					// HttpCustomBasicUnauthorizedResult inherits from HttpUnauthorizedResult and does the
    					// work of displaying the basic authentication prompt to the client
    					filterContext.Result = new HttpCustomBasicUnauthorizedResult();
    				}
    			}
    		}
    
    		// from here on are private methods to do the grunt work of parsing/verifying the credentials
    
    		private bool Authenticate(HttpContextBase context)
    		{
    			if (_RequireSsl && !context.Request.IsSecureConnection && !context.Request.IsLocal) return false;
    
    			if (!context.Request.Headers.AllKeys.Contains("Authorization")) return false;
    
    			string authHeader = context.Request.Headers["Authorization"];
    
    			IPrincipal principal;
    			if (TryGetPrincipal(authHeader, out principal))
    			{
    				HttpContext.Current.User = principal;
    				return true;
    			}
    			return false;
    		}
    
    		private bool TryGetPrincipal(string authHeader, out IPrincipal principal)
    		{
    			var creds = ParseAuthHeader(authHeader);
    			if (creds != null)
    			{
    				if (TryGetPrincipal(creds[0], creds[1], out principal)) return true;
    			}
    
    			principal = null;
    			return false;
    		}
    
    		private string[] ParseAuthHeader(string authHeader)
    		{
    			// Check this is a Basic Auth header
    			if (authHeader == null || authHeader.Length == 0 || !authHeader.StartsWith("Basic")) return null;
    
    			// Pull out the Credentials with are seperated by ':' and Base64 encoded
    			string base64Credentials = authHeader.Substring(6);
    			string[] credentials = Encoding.ASCII.GetString(Convert.FromBase64String(base64Credentials)).Split(new char[] { ':' });
    
    			if (credentials.Length != 2 || string.IsNullOrEmpty(credentials[0]) || string.IsNullOrEmpty(credentials[0])) return null;
    
    			// Okay this is the credentials
    			return credentials;
    		}
    
    		private bool TryGetPrincipal(string userName, string password, out IPrincipal principal)
    		{
    			// this is the method that authenticates against my repository (in this case, hard coded)
    			// you can replace this with whatever logic you'd use, but proper separation would put the
    			// data access in a repository or separate layer/library.
    			UserBase user = Repository.Authenticate(userName, password);
    
    			if (user != null)
    			{
    				// once the user is verified, assign it to an IPrincipal with the identity name and applicable roles
    				principal = new GenericPrincipal(new GenericIdentity(user.UserName), user.Roles);
    				return true;
    			}
    			else
    			{
    				principal = null;
    				return false;
    			}
    		}
    	}
    

I also added a new subclass called HttpCustomBasicUnauthorizedResult. This just kept the code cleaner, IMO, but the real important part is making sure that the "WWW-Authenticate" header gets passed back with the response. This, in combination with the 401 HTTP Error that is assigned by the base class, is what makes the client know that basic authentication is being required.

    public class HttpCustomBasicUnauthorizedResult: HttpUnauthorizedResult
    	{
    		// the base class already assigns the 401.
    		// we bring these constructors with us to allow setting status text
    		public HttpCustomBasicUnauthorizedResult() : base() { }
    		public HttpCustomBasicUnauthorizedResult(string statusDescription) : base(statusDescription) { }
    
    		public override void ExecuteResult(ControllerContext context)
    		{
    			if (context == null) throw new ArgumentNullException("context");
    
    			// this is really the key to bringing up the basic authentication login prompt.
    			// this header is what tells the client we need basic authentication
    			context.HttpContext.Response.AddHeader("WWW-Authenticate", "Basic");
    			base.ExecuteResult(context);
    		}
    	}
    

## Putting It To Use

For this sample app, we'll create a simple AccountController with two methods. One, called Authenticate, will not be secured (so we can get authenticated) and the other, called Me, will require authentication and display information about the logged in user.

    public class AccountController : Controller
    	{
    
    		// this is only necessary if you are doing dependency injection
    		IUserRepository _UserRepo;
    		public AccountController(IUserRepository userRepo)
    		{
    			_UserRepo = userRepo;
    		}
    		// end DI code
    
    		public ActionResult Authenticate()
    		{
    			return Json("This is open for unauthenticated access.", JsonRequestBehavior.AllowGet);
    		}
    
    		[CustomBasicAuthorize]
    		public ActionResult Me()
    		{
    			return Json(_UserRepo.GetDetails(User.Identity.Name), JsonRequestBehavior.AllowGet);
    		}
    
    	}
    

And in this case, the repository (as seen below) is just validating against or supplying some hard coded values. In this case, the username/password combination of "user" and "pass" will get you logged in and then once you are logged in, it will display those same hard coded values as JSON.

    public class HardCodedUserRepository: IUserRepository
    	{
    		public UserBase Authenticate(string userName, string password)
    		{
    			// you would likely query the database here instead...
    			if (userName.ToLowerInvariant() == "user" && password == "pass")
    				return new UserBase { Id = "1", UserName = "user", FirstName = "Some", LastName = "User", Email = "user@user.com", Roles = new string[] { "Admin", "Manager" } };
    			else
    				return null;
    		}
    
    		public UserBase GetDetails(string userName)
    		{
    			// you would likely query the database here instead...
    			if (userName.ToLowerInvariant() == "user")
    				return new UserBase { Id = "1", UserName = "user", FirstName = "Some", LastName = "User", Email = "user@user.com", Roles = new string[] { "Admin", "Manager" } };
    			else
    				return null;
    		}
    	}
    

[![][7]][7]
Authenticated JSON Output

The nice thing about this model is that since it uses an IPrincipal, all the parts of authentication you are probably already used to still work. You can use User.IsInRole() for manual testing of privileges or you can add properties to your attribute to only allow certain users or roles to specific actions or controllers. For example, an attribute like:

`[CustomBasicAuthorize(Users="user, otheruser", Roles="admin, manager")]`

will allow only the specified users and roles to access whatever action/controller it is decorating.

## Wrapping Up

So, with only two classes (not counting any of the repository or dependency injection code), we've now created a very extensible way to easily secure any/all of our API methods so that any client can now pass credentials using a standardized protocol!

One thing that I hope goes without saying is that if you are using Basic Authentication for anything, it should **always** be done over SSL. Even though the credentials are Base64 encoded, they are still just plain text and can be easily decoded and viewed by anyone inspecting the transmission (it only took us one line of code to decode it above).

The full source code for this project is available here: Download the code (CustomizedBasic.zip) on [GitHub][7].

Happy Coding!

 [1]: http://oauth.net/
 [2]: http://stackoverflow.com/questions/1407153/rest-ful-basic-authentication-with-asp-net-mvc
 [3]: http://blogs.msdn.com/b/astoriateam/archive/2010/07/21/odata-and-authentication-part-6-custom-basic-authentication.aspx
 [4]: http://twitter.com/adjames
 []: http://cacheandquery.com/blog/wp-content/uploads/2011/03/iis-auth.png
 []: http://cacheandquery.com/blog/wp-content/uploads/2011/03/screenshot.png
 [7]: https://github.com/jslaybaugh/MvcCustomizedBasicAuthentication  

