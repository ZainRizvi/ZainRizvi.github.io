---
title: "Backup just the Important Parts of your Site with Azure Web Apps"
---

Introducting a way to backup just the parts of your website that matter most.

# Introduction

Azure Web Apps provides powerful [backup/restore functionality](https://azure.microsoft.com/en-us/documentation/articles/web-sites-backup/). (Because disasters can [happen to anyone](http://blog.codinghorror.com/international-backup-awareness-day/))

However, sometimes you don't want to backup everything on your site, especially if you backup your site regularly, or if your site has over 10GB of content (that's the max amount you can backup at a time).

For example, you probably don't want to back up the log files. Or if you [setup weekly backups](https://azure.microsoft.com/en-us/documentation/articles/web-sites-backup/#configure-automated-backups) you won't want to fill up your storage account with static content that never changes like old blog posts or images. 

Partial backups will let you choose exactly which files you want to back up.

# Specify the files you don't want to backup

You can create a list of files and folders to exclude from the backup.  

You save the list as a text file called _backup.filter in the wwwroot folder of your site. An easy way to access this is through the [Kudu Console](https://github.com/projectkudu/kudu/wiki/Kudu-console) at `http://{yoursite}.scm.azurewebsites.net/DebugConsole`.  

The instructions below will be using the Kudu Console to create the _backup.filter file, but you can use your favorite deployment method to put the file there.

## What to do

I've got a site that contains log files and static images from past years that are never going to change.

I already have a full backup of the site which includes the old images. Now I want to backup the site every day, but I don't want to pay for storing log files or the static image files that never change.

![Log files directory](/images/2015/06/Logs-1.PNG) ![Images directory](/images/2015/06/Images-2.PNG)

The below steps show how I'd exclude those files from the backup.

### Identify the files and folders you don't want to backup

This is easy. I already know I don't want to backup any log files, so I want to exclude `D:\home\site\wwwroot\Logs`.  

There's another log file folder that all Azure Web Apps have at `D:\home\LogFiles`. Let's exclude that too.

I also don't want to backup the images from previous years over and over again. So lets add `D:\home\site\wwwroot\Images\2013` and `D:\home\site\wwwroot\Images\2014` to the list as well.

Finally, let's not backup the brand.png file in the Images folder either, just to show we can blacklist individual files as well. It's located at `D:\home\site\wwwroot\Images\brand.png` 

This gives us the following folders that we don't want to backup:

* D:\home\site\wwwroot\Logs
* D:\home\LogFiles
* D:\home\site\wwwroot\Images\2013
* D:\home\site\wwwroot\Images\2014
* D:\home\site\wwwroot\Images\brand.png

### Create the exclusion list

You save the blacklist of files and folders that you don't want to backup in  a special file called _backup.filter.  Create the file and place it at `D:\home\site\wwwroot\_backup.filter`.

List all the files and folders you don't want to backup in the _backup.filter file. You add the full path relative to D:\home of the folder or file that you want to exclude from the backup, one path per line.

So for my site, `D:\home\site\wwwroot\Logs` becomes `\site\wwwroot\Logs`, `D:\home\LogFiles` becomes `\LogFiles`, so on and so forth, resulting in the following contents for my _backup.filter:

	\site\wwwroot\Logs
    \LogFiles
    \site\wwwroot\Images\2013
    \site\wwwroot\Images\2014
    \site\wwwroot\Images\brand.png

Note the starting `\` at the beginning of each line. That's important.

# Run a backup

Now you can run backups the same way you would normally do it. [Manually](https://azure.microsoft.com/en-us/documentation/articles/web-sites-backup/#create-a-manual-backup), [automatically](https://azure.microsoft.com/en-us/documentation/articles/web-sites-backup/#configure-automated-backups), either way is fine.

Any files and folders that fall under the filters listed in the _backup.filter will be excluded from the backup. This means now the log files and the 2013 and 2014 image files will no longer be backed up.

# Restoring your backed up site

You restore partial backups of your site the same way you would [restore a regular backup](https://azure.microsoft.com/en-us/documentation/articles/web-sites-restore/). It'll do the right thing.

#### The technical details

With full (non-partial) backups normally all content on the site is replaced with whatever is in the backup.  If a file is on the site but not in the backup it gets deleted.

But when restoring partial backups though any content that is located in one of the blacklisted folders (like `D:\home\site\wwwroot\images\2014` for my site) will be left as is. And if individual files were black listed then they'll also be left alone during the restore.

# Best Practices

What do you do when disaster strikes and you have to restore your site?  Make sure you're prepared beforehand.

Yeah, you have partial backups, but take at least one full backup of the site first so that you have all your site's contents backed up (this is worst case scenario planning).  Then when you're restoring your backups you can first restore the full backup of the site, and then restore the latest partial backup on top of it.

Here's why: it lets you use [Deployment Slots](https://azure.microsoft.com/en-us/documentation/articles/web-sites-staged-publishing/) to test your restored site. You can even test the restore process without ever touching your production site. And testing your restore process is a [Very Good Thing](http://axcient.com/blog/one-thing-can-derail-disaster-recovery-plan/).  You never know when you might run into some subtle gotcha like I did when I tried restoring my blog and end up losing half your content.

## My horror story

My blog is powered by the [Ghost](https://ghost.org/) blogging platform.  Like a responsible dev I created a backup of my site and everything was great. Then one day I got a message saying that there was a new version of Ghost available and I could upgrade my blog to it. Great!

I created one more backup of my site to backup the latest blog posts, and proceeded to upgrade Ghost. 

On my production site. 

Bad mistake.  

Something went wrong with the upgrade, my home screen just showed a blank screen.  "No problem" I thought, "I'll simply restore the backup I just took."

I restored the upgrade, saw everything come back...except the blog posts.

WHAT???

Turns out, in the [Ghost upgrade notes](http://support.ghost.org/how-to-upgrade/) there's this warning:

![You can take a copy of your database from content/data but you  should not do this while Ghost is runing. Please stop it first](/images/2015/06/Ghost--upgrade-warning.PNG)

If you try to backup the data while Ghost is running...the data doesn't actually get backed up.

Bummer.

If I had tried the restore on a test slot first I would have seen this issue and not lost all my posts.

Such is life. It can happen to [the best of us](http://blog.codinghorror.com/international-backup-awareness-day/).  

