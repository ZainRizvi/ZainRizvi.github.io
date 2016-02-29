---
title: "Deploy statically generated sites to Azure Web Apps"
---
There are a lot of awesome static site generators out there.  It's not always easy to figure out how to setup continuous deployment for them though.


This post will describe how to deploy a statically generated site using yeoman angular to Azure Web Apps, but these steps can be applied to deploy any statically generated site to Azure Web Apps.

---

#### Deploying the initial site
I tried using yeoman's gulp-angular generator.  I made a quick site following their [tutorial](http://yeoman.io/codelab/index.html), setup [continuous deployment via github](https://azure.microsoft.com/en-us/documentation/articles/web-sites-publish-source-control/), navigated to the newly deployed site and I saw...huh?

![img](/images/2015/10/01-No-permission-to-site.png)

What's going on here? 

#### Using Kudu for debugging
Luckily all Azure Web Apps come with a handy [Kudu site](https://github.com/projectkudu/kudu/wiki) that gives you command line access to your site. You can get to it at `https://<yourSiteName>.scm.azurewebsites.net\DebugConsole`.

![alt](/images/2015/10/02-Kudu-console.png)

I navigated to the site's `D:\home\site\wwwroot` folder and saw all the content was there.  And that's when I face-palmed and realized *that statically generated site is saved to the `dist` folder, and that's not even part of the deployment!!!*

Luckily, that's easy enough to fix.

#### Check the static site into the source code
First issue was to include the `dist` folder in the source code. You just need to exclude the `/dist` line from the .gitignore file for that. [Easy enough](https://github.com/ZainRizvi/YoAngularOnAzureWebApps/commit/3fc3040eb65699295e85c151f339dc30aae6c971#diff-a084b794bc0759e7a6b77810e01874f2).

Now when you deploy your site to Azure Web Apps your site exists in the new `D:\home\site\wwwroot\dist` folder!

![alt](/images/2015/10/03-Dist-folder-appears.png)
(Fyi, with `yo angular` you have to run `grunt` once first before you check in your code to actually generate the `dist` folder).

But your site still doesn't work...because Azure Web Apps is expecting the site's content to be in `D:\home\site\wwwroot`.  

Darn.

#### Custom deployment settings to the rescue!

Add a `.deployment` [file](https://github.com/ZainRizvi/YoAngularOnAzureWebApps/commit/9be9a4b503a86678d85e3a4287fa26cce1f175b7) to the root folder of your code and paste the below inside:

    [config]
    project = dist

This will tell Azure Web Sits that the root folder for your site is the `dist` folder. Now your sites will be hosted from the `D:\home\site\wwwroot\dist` folder. If your static site generator puts your site in some other folder, set project to that folder's name. 

Check in the file, deploy it to Azure Web Apps, and see the magic happen. 

![alt](/images/2015/10/04-Working-site.png)

You can find a full copy of the sample code here on Github, with check-ins corresponding to each step of this tutorial: https://github.com/ZainRizvi/YoAngularOnAzureWebApps 

You can see the final working site here: http://yoangularonazurewebapps.azurewebsites.net