---
title: "Create a site using Facebook authentication on ASP.NET Core"
---

The goal of this post is to setup Facebook authentication in a way that we get access to the person's basic information and their email address when they log in.

Setting up auth from an external service is hard.  At least it is if the steps aren't really documented.  I'm sharing the steps I followed to get Facebook auth working for a brand new ASP.NET Core site.

These instructions assume you've got Visual Studio 2015 RC1 installed. I expect these steps should keep working after VS RTMs as well.

# Getting started

The [official docs](http://docs.asp.net/en/latest/security/authentication/sociallogins.html) contain good instructions, but they're not quite complete.  Let's start off by following them and then we'll move forward from there:

1. [Create a new ASP.NET project](http://docs.asp.net/en/latest/security/authentication/sociallogins.html#use-secretmanager-to-store-facebook-appid-and-appsecret)

2. [Run the app to make sure it works](http://docs.asp.net/en/latest/security/authentication/sociallogins.html#running-the-application). In the site, click the "Log in" button on the top left corner to see the login screen.  You'll note that it lets you create a new account.  It saves this account in a local Sql database that was installed on your computer with Visual Studio (we'll talk more about this later). On the right hand side you'll see instructions for setting up logins using an external service like Facebook.

   ![Login window](/images/2016/03/Login-Screen.png)

3. [Get Facebook developer credentials by creating an app in Facebook](http://docs.asp.net/en/latest/security/authentication/sociallogins.html#creating-the-app-in-facebook)

4. [Use SecretManager to store the Facebook AppId and AppSecret](http://docs.asp.net/en/latest/security/authentication/sociallogins.html#use-secretmanager-to-store-facebook-appid-and-appsecret). The SecretManager is a pretty cool tool since it gives you a good place to put your secrets without risking them accidentally ending up checked into your source control and ending up on Github

5. [Enable the Facebook middleware](http://docs.asp.net/en/latest/security/authentication/sociallogins.html#enable-facebook-middleware). This tells ASP.NET Core that you want to use Facebook authentication and will set you up with the basics that  you'll need.  Visual Studio will prompt you to add a reference to Microsoft.AspNet.Authentication.Facebook when you add the Facebook middleware code.
  
Herea are the two changes you'll have made to your project:  [adding the Facebook middleware](https://github.com/ZainRizvi/ASP.NET-Core-FacebookAuth/commit/c2472e966c0d24262280691b53ae039ba341eb51) and [adding the reference](https://github.com/ZainRizvi/ASP.NET-Core-FacebookAuth/commit/64d0f89e6644561b154e869c59b19da4f9072534)

# Now for the custom stuff

## Setting up the database 

If you run your site and try to login with your Facebook account you may notice the following error:

![Login window](/images/2016/03/Database-failure.png)

That's because we never setup the database!  What, you don't remember creating a database? That's because it was done for you automatically when you created the project, or at least part of it was. 

When you installed Visual Studio 2015, you also got a copy of sqllocaldb installed on your computer. That lets your computer act as a local Sql Server instance and you can create and access databases on it.  If you look in your site's appsettings.json file you'll see something similar to the following setting that sets your connection string:

``` json
 "Data": {
    "DefaultConnection": {
      "ConnectionString": "Server=(localdb)\\mssqllocaldb;Database=aspnet5-FacebookAuthSite-1746853f-78c4-4424-8598-a8e8d950b167;Trusted_Connection=True;MultipleActiveResultSets=true"
    }
  }
```
  
That's says where your database is.  This is great for testing, but once you're ready to host your site in the cloud you'll want to change that connection string to point to a database on Azure SQL Server or some other cloud server. You don't need to worry abou this right now, but when you're ready to change the connection string you can use the [SecretManager](http://docs.asp.net/en/latest/security/authentication/sociallogins.html#use-secretmanager-to-store-facebook-appid-and-appsecret) on your local computer to hold the connection string for the local sql db, while use a different type of configuration (like an App Setting if you're hosting your site on Azure Web Apps) to store your production database's connection string.

We need to create the all the user tables for our site (we never created them, hence the error message that we saw). Normally we'd do this using Entity Framework migrations on the command line (via commands 'dnx ef migrations add {migrationName}' and 'dnx ef migrations update'), but luckily for us our site shows a handy "Apply Migrations" button that'll just do that for us.  Just click that button and your database will be setup. And then hit refresh to see the working web page.

# Getting access to the User's Email

Now if you go back to the login screen and try to create a site using Facebook authentication, you'll be redirected to a page that prompts you to add an email address to complete your registration.  

![Login window](/images/2016/03/Register-user.png)

That's a bit inconvenient for the user, ideally we'd just ask Facebook to give us the user's email address. So let's do that.

The way we do that is by asking the user for permission to access their personal information when we register them.  This is done by specifying [Facebook Permissions](https://developers.facebook.com/docs/facebook-login/permissions) in our authentication code. Each kind of permission lets us pull different user information from Facebook.  We really just want 'email', but let's also get 'public_profile' just for the heck of it (I think we get 'public_profile' by default anyways).

We add these permissions by to the options we send in to the UseFacebookAuthentication middleware.  In Startup.cs, in the Configure method, change the call to UseFacebookAuthentication to:

``` csharp
app.UseFacebookAuthentication(options =>
{
    options.AppId = Configuration["Authentication:Facebook:AppId"];
    options.AppSecret = Configuration["Authentication:Facebook:AppSecret"];

    options.Scope.Add("public_profile");
    options.Scope.Add("email");
}); 
```

Now when the user tries to register using Facebook auth they'll see something like this if they click the 'Edit the info you provide' button on Facebook. 

![Login window](/images/2016/03/Facebook-info.png)

When they click Ok you'll have access to read their email address!

# Actually reading the email address

Great, we can access the email address, but when we try to register we still get back to the same page as before with a blank email:

![Login window](/images/2016/03/Register-user.png)

What gives?  Let's take a look at the code. The route we're hitting is "Account/ExternalLoginCallback", so let's look at the Account controller class, ExternalLoginCallback method. Near the bottom of that method is the code

``` csharp
var email = info.ExternalPrincipal.FindFirstValue(ClaimTypes.Email);
```

So we're expecting the email address claim ([claims](http://dotnetcodr.com/2013/02/11/introduction-to-claims-based-security-in-net4-5-with-c-part-1/) are how we identify information about users who are trying to log in) but we're not getting it. If you inspect that code in the debugger you can verify that there is no email claim.

Turns out Facebook doesn't send that to us when the user logs in. I don't know why. But we can explicitly ask Facebook for this information.  To do that we need to make an http request to [Facebook's Graph Api](https://developers.facebook.com/docs/graph-api). The Graph Api lets us access all the information about the user that we had earlier requested access to.

The Graph Api is a REST Api, but in order to avoid the complexities of using it I'll just use the Facebook nuget package. Unfortunately this package does not work on the dnxcore environment, meaning we can only run it on Windows based servers, but you can create your own Facebook Graph Api client if you want to run your site using dnxcore.

Let's install the Facebook nuget package by modifying the frameworks element in your project.json file to like like the following:

``` json
"frameworks": {
  "dnx451": {
    "dependencies": {
      "Facebook": "7.0.6"
    }
  }
}
```

We had to remove the dnxcore framework since we will no longer support it, and we're marking the Facebook package explicitly as a dnx451 only package.

Now to actually use the Graph Api lets go back to the Startup.cs class -> Configure method -> UseFacebookAuthentication middleware.  We want to configure the options to say that whenever a user logs in, call the Facebook Graph Api and get the user's email address, and add that as the email claim.  You'll use the following code to do that (intellisense will prompt you to add some dependency references and using statements as well):

``` csharp
app.UseFacebookAuthentication(options =>
{
    options.AppId = Configuration["Authentication:Facebook:AppId"];
    options.AppSecret = Configuration["Authentication:Facebook:AppSecret"];

    options.Scope.Add("public_profile");
    options.Scope.Add("email");

    options.Events = new OAuthEvents
    {
        OnCreatingTicket = context => {
            // Use the Facebook Graph Api to get the user's email address
            // and add it to the email claim

            var client = new FacebookClient(context.AccessToken);
            dynamic info = client.Get("me", new { fields = "name,id,email" });

            context.Identity.AddClaim(new Claim(ClaimTypes.Email, info.email));
            return Task.FromResult(0);
        }
    };
});
```

(Fyi, the team managing the Facebook package has vanished into the netherwebs, but you can find a [cached copy of their documentation here] (http://web.archive.org/web/20150317045522/http://facebooksdk.net/docs/web/getting-started/))

Now if we redeploy our site and try to register using Facebook we'll see the following page:

![Login window](/images/2016/03/Register-user-with-email.png)

We have the email!

Note, this message is a bit wasteful since we'll be asking Facebook to give us the user's email address on _every_ authenticated request that they make, when we really only want to do this when they first register.  I'll leave it as an exercise for the user to optimize this code flow.

The code you have now should match what's in [this commit](https://github.com/ZainRizvi/ASP.NET-Core-FacebookAuth/commit/f771c75cc89691504a04e9b5b985428998002f8e).

# Getting the User Name from Facebook and getting rid of the extra Registration page

Great, we're now reading the email address.  And we know the user's name. So what do we need this extra registration page for?

![Login window](/images/2016/03/Register-user-with-email.png)

We should really just register the user automatically.

Digging around in the code, we see that in the AuthenticationController class, the ExternalLoginCallback method creates an ExternalLoginConfirmationViewModel view model with the user's email and then calls the ExternalLoginConfirmation view to let the user add in the custom details, which, when the user hits "Register" will send a POST request to the ExternalLoginConfirmation method with all the required data. We can skip the middle step of involving the user and just send the required data to the  ExternalLoginConfirmation method ourselves. 

In fact, if you look at the ExternalLoginConfirmation method, they set the user's email as the user name as well. Let's fix that too while we're at it.

``` csharp
var user = new ApplicationUser { UserName = model.Email, Email = model.Email };
```

The first step will be to extend our ExternalLoginConfirmationViewModel to add a UserName property.  Edit ViewModels\ExternalLoginConfirmationViewModel.cs to look like the following:

``` csharp
public class ExternalLoginConfirmationViewModel
{
    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [Required]
    [EmailAddress]
    public string UserName { get; set; }
}
```

Now, in our AccountController's ExternalLoginCallback method, change the last else statement (where it calls the ExternalLoginConfirmation View) to match the below code. We're specifying the UserName using the Name claim and passing that directly to the ExternalLoginConfirmation method instead of invoking a View.  The ExternalLoginConfirmation will generate and return the regisration completed view directly.

``` csharp
else
{
    // If the user does not have an account, then ask the user to create an account.
    ViewData["ReturnUrl"] = returnUrl;
    ViewData["LoginProvider"] = info.LoginProvider;
    var email = info.ExternalPrincipal.FindFirstValue(ClaimTypes.Email);
    var name = info.ExternalPrincipal.FindFirstValue(ClaimTypes.Name);

    return await ExternalLoginConfirmation(new ExternalLoginConfirmationViewModel { Email = email, UserName = name });
    name });
}
```

We also need to modify the ExternalLoginConfirmation to actually read the UserName from the model:

``` csharp
public async Task<IActionResult> ExternalLoginConfirmation(ExternalLoginConfirmationViewModel model, string returnUrl = null)
{
    ...

    if (ModelState.IsValid)
    {
        // Get the information about the user from the external login provider
        var info = await _signInManager.GetExternalLoginInfoAsync();
        if (info == null)
        {
            return View("ExternalLoginFailure");
        }
        var user = new ApplicationUser { UserName = model.UserName, Email = model.Email };
        var result = await _userManager.CreateAsync(user);
```

And finally we need to modify the last line of ExternalLoginConfirmation, where it returns the view, to explicitly mention the name of the view:

``` csharp
return View("ExternalLoginConfirmation", model);
```

Without that line, when ExternalLoginCallback calls ExternalLoginConfirmation, the View method would look for a view named ExternalLoginCallback, which doesn't exist.

Now let's try to run our site and log in.  And you might now see an error like 

![Invalid user name](/images/2016/03/Invalid-user-name.png)

The message says that the name can only have numbers and letters, but my name contains a space so it's complaining. It's gonna be tough getting names from Facebook if we don't allow spaces in the name, so let's fix that.

This setting resides back in Startup.cs, but this time we'll go to the ConfigureServices method, which is where all the dependency injection gets setup.  We'll modify the ```services.AddIdentity()``` method to pass in the below options:

``` csharp
services.AddIdentity<ApplicationUser, IdentityRole>(options => {
    options.User.AllowedUserNameCharacters = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789-._' ";
})
.AddEntityFrameworkStores<ApplicationDbContext>()
.AddDefaultTokenProviders();
```

This expands our list of allwed characters in the user name to include a space, and I also threw in a few extra characters for good measure.  It's not perfect, but it'll at least protect us from some of the [falsehoods developers believe about names](http://www.kalzumeus.com/2010/06/17/falsehoods-programmers-believe-about-names/).

Try running your site again and registering as a new user using Facebook authentication and...hey we're in!

![We're logged in](/images/2016/03/Logged-in.png)

Fyi, [this commit](https://github.com/ZainRizvi/ASP.NET-Core-FacebookAuth/commit/0691694a15f38c7fb034feff0749724c9c60e25e) contains all the changes you should have made.

# Poking around the database

We've registered a user, but where is the data actually stored? Lets take a quick look around the database and check it out.

In Visual Studio, go to View->SQL Server Object Explorer to start browsing the database.  There if you poke around the folders you'll find a database corresponding to your project.

![Browsing SQL Server Object Explorer](/images/2016/03/Local-Sql-Server.png)

You can expand the database to see all the tables in there. Right now you'll only see tables created for managing users.  

![Browsing SQL Server Tables](/images/2016/03/Sql-Server-Tables.png)

Right click the dbo.AspNetUsers table and click "View data", and you'll see a table open up containing the Facebook user you just registered!

# The End

That's it for this post. Hope you guys were able to follow all the steps.  Shout out in the comments if you found this stuff helpful or if you run into any problems. 