---
layout: post
title: "How to Redirect the default *.azurewebsites.net domain to your custom domain on Azure Web Apps"
excerpt: "What if you add a custom host name to your Azure Web App and don't want people to be able to access the default *.azurewebsites.net domain anymore?"
tags: [Tutorial]
---

When you create a new website using [Azure Web Apps](https://azure.microsoft.com/en-us/services/app-service/web/) you get a default `<sitename>.azurewebsites.net` domain assigned to your site.  That's great, but what if you add a custom host name to your site and don't want people to be able to access your default *.azurewebsites.net domain anymore? (You paid good money for that custom domain.)  This post explains how to redirect all traffic aimed at your site's default domain to your custom domain instead.

It's really simple. You just need to add a redirect rule to your site's web.config file.  You can do that by adding the following rewrite rule to the web.config file in your wwwroot folder.  If you don't have a web.config file, then you can create one and just paste the text below into it, just change the host names to match your site's host names:

```
<configuration>
  <system.webServer>  
    <rewrite>  
        <rules>  
          <rule name="Redirect rquests to default azure websites domain" stopProcessing="true">
            <match url="(.*)" />  
            <conditions logicalGrouping="MatchAny">
              <add input="{HTTP_HOST}" pattern="^yoursite\.azurewebsites\.net$" />
            </conditions>
            <action type="Redirect" url="http://www.yoursite.com/{R:0}" />  
          </rule>  
        </rules>  
    </rewrite>  
  </system.webServer>  
</configuration>  
```

Basically we're telling IIS to take any request where the host name matches the RegEx pattern "^yoursite\.azurewebsites\.net$" and return an HTTP 301 response. The response will include the originally requested url, except it'll be pointing to your custom "www.yoursite.com" domain instead.  When the user's browser reads that 301 response and the new url, it will automatically load that new url instead. It'll even change the address the user sees in the address bar.

That's great, so how do we parse the above code exactly?  I'm not a fan of copying code unless I know exactly what it's doing. 

So let's see what's going on here:

```
<configuration>
  <system.webServer>  
  ...
  </system.webServer>  
</configuration>  
```

This part tells IIS that we're modifying the web server's configuration settings. The next section is where it starts to get tricky:

```
<rewrite>  
    <rules> 
    ... 
    </rules>  
</rewrite>  
```

The `<rewrite>` tag tells IIS that the elements it encloses are settings for the the [URL Rewrite](http://www.iis.net/learn/extensions/url-rewrite-module/creating-rewrite-rules-for-the-url-rewrite-module) module.  The `<rules>` tag lists all the rules that we want that module to follow.   In our case, we want it to follow a rule that will return a HTTP 301 redirect response to the client (the user's web browser).

Now for the actual rule:

```
<rule name="Redirect rquests to default azure websites domain" stopProcessing="true">
  <match url="(.*)" />  
  <conditions logicalGrouping="MatchAny">
    <add input="{HTTP_HOST}" pattern="^yoursite\.azurewebsites\.net$" />
  </conditions>
  <action type="Redirect" url="http://www.yoursite.com/{R:0}" />  
</rule>  
```

The "name" attribute is just for human readability. It doesn't affect execution at all. StopProcessing="true" tells the rewrite module that if this rule applies to the incomming request, then not to bother processing any other rules  after this one becuase they won't matter.  In our case we only have one rule, so this tag doesn't do anything, but it can save you some CPU if you have more rules defined.

Next is the `<match url="(.*)" />` section. That's a regEx pattern inside, and since this regEx covers all possible inputs, it tells IIS to apply this rule to all requests, no matter what their url is.

Then comes the conditions section. We set `logicalGrouping="MatchAny"` to tell IIS to execute the rule if any of the following conditions hold true.  Right now we only have one condition, so again it doesn't matter, but if you had multiple conditions (for example, multiple domain names you wanted to forward to your custom domain name) then you could list them all here. Alternatively, you could set it to "MatchAll" to tell IIS to only run the action if it matches all the conditons given.

Here's the condition we used:

```
<add input="{HTTP_HOST}" pattern="^yoursite\.azurewebsites\.net$" />
```

It says to look at the http host and evaluate the condition as true if the host matches the given regEx pattern, which we set to your default azure domain name.

The last bit is the action, the meat of the whole rule:

```
<action type="Redirect" url="http://www.yoursite.com/{R:0}" />  
```

Here we're telling the Rewrite module what action to take (what it's actually supposed to do) when a request matches the above rules.  The action we want is "Redirect", which is where it'll return the HTTP 301 to the client, and when it returns a 301 we need to tell the client what url it should be redirected to. That's how we get to specify our desired domain name. 

But we don't want to send the user to the root of the domain name, so we add in the `/{R:0}` bit, which (put simply) says "Look at the orignal url we matched against in the `<match>` tag, and stick that in."  A more thorough description is that we ran a regex expression in the `<match>` tag, and this `{R:0}` returns the first RegEx group.

And there you have it, that's how you can redirect all request for your default azure domain to your custom domain.

As a final example, here's web.config file's conent for my site:

```
<configuration>
  <system.webServer>  
    <rewrite>  
        <rules>  
          <rule name="Redirect rquests to default azure websites domain" stopProcessing="true">
            <match url="(.*)" />  
            <conditions logicalGrouping="MatchAny">
              <add input="{HTTP_HOST}" pattern="^zainrizvi\.azurewebsites\.net$" />
            </conditions>
            <action type="Redirect" url="http://www.zainrizvi.io/{R:0}" />  
          </rule>  
        </rules>  
    </rewrite>  
  </system.webServer>  
</configuration>  
```

