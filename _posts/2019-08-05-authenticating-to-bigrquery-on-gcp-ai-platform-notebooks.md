---
layout: post
title: 'Authenticating to BigRQuery on GCP AI Platform Notebooks '
excerpt: When you're using Jupyter Notebooks with R, authenticating with BigQuery
  can be tricky.  Here's how to do it.
tags:
- BigRQuery
- BigQuery
- Jupyter Lab
- R
- Google Cloud Platform

---
Note: These instructions can be used whenever you're using Jupyter Lab on a remote machine

[GCP AI Platform Notebooks](https://cloud.google.com/ai-platform-notebooks/) makes it really easy to run notebooks on Jupyter Lab and even offers R language notebooks.  R is great for crunching large data sets, and a popular place for people to store their data is on BigQuery.

The most popular library for accessing BigQuery in R is the open source library [BigRQuery](https://github.com/r-dbi/bigrquery).  It's an extremely useful library, but it has the downside that the authentication step will try one of two things:

1. It either will prompt people for extra input on the command line
2. Or open up a port on http://localhost to listen for GCP authentication

\#1 doesn't work with any Jupter  Notebook since they are not designed to accept extra commands in the middle of an execution

\#2 won't work if you're connected to Jupyter Lab on a remote machine (the http://localhost will point you to the wrong VM!)

Since neither of the normal ways to authenticate yourself will work [just yet](https://github.com/r-dbi/bigrquery/issues/340), this post describes two different methods you can use to authenticate yourself to BigQuery within a AI Platform Notebook:

1. Authenticate using your normal GCP user credentials
2. Authenticate using a service account

# Prereq: Create an AI Platform Notebook for R

First create a new AI Platform notebook. This notebook is where you'll be trying to use BigRQuery

1. Go to [http://console.cloud.google.com/mlengine/notebooks/instances](http://console.cloud.google.com/mlengine/notebooks/instances "http://console.cloud.google.com/mlengine/notebooks/instances")
2. Select 'New Instance' -> 'R 3.x' -> Create
3. Wait for the notebook to be created and click "OPEN JUPYTERLAB"

# Option #1: Authenticating using your GCP user credentials

This method uses the Jupter Lab terminal to run the interactive commands and cache the credentials. Once you're authenticated, you can switch to a notebook and it'll use the credentials in the cache.

First, start R in a Terminal

![](/media/2019-08-05-b1.png)

Run `R`

You'll get the output:

    jupyter@r-20190802-172922:\~$ R
    
    R version 3.6.1 (2019-07-05) -- "Action of the Toes"
    Copyright (C) 2019 The R Foundation for Statistical Computing
    Platform: x86_64-pc-linux-gnu (64-bit)
    
    R is free software and comes with ABSOLUTELY NO WARRANTY.
    You are welcome to redistribute it under certain conditions.
    Type 'license()' or 'licence()' for distribution details.
    
    Natural language support but running in an English locale
    
    R is a collaborative project with many contributors.
    Type 'contributors()' for more information and
    'citation()' on how to cite R or R packages in publications.
    
    Type 'demo()' for some demos, 'help()' for on-line help, or
    'help.start()' for an HTML browser interface to help.
    Type 'q()' to quit R.

Next we install the required packages

As of this writing, BigRQuery needs the dev version of gargle for this authentication to work.  Later you shouldn't need to explicitly install gargle.

Run the following commands to install the packages:

    install.packages("httpuv")
    install.packages("devtools")
    devtools::install_github("r-lib/gargle") # get the dev version of gargle
    install.packages("bigrquery")
    install.packages("readr") # To read BigQuery results

Those packages will take \~10 minutes to install

Import the required libraries and Authenticate yourself by running the command `bq``_auth(use_``oob = TRUE)` (correct your email address as appropriate)

Commands to run:

    library(httpuv)
    library(gargle)
    library(bigrquery)
    bq_auth(use_oob = TRUE)

Say yes when it asks about caching the OAuth credentials.

You'll see an error like the following

    > library(httpuv)
    > library(gargle)
    > library(bigrquery)
    > bq_auth(use_oob = TRUE)
    > Is it OK to cache OAuth access credentials in the folder '/home/jupyter/.R/gargle/gargle-oauth' between R sessions?
    
    1: Yes
    2: No
    
    Selection: 1
    Enter authorization code: /usr/bin/xdg-open: 778: /usr/bin/xdg-open: www-browser: not found
    /usr/bin/xdg-open: 778: /usr/bin/xdg-open: links2: not found
    /usr/bin/xdg-open: 778: /usr/bin/xdg-open: elinks: not found
    /usr/bin/xdg-open: 778: /usr/bin/xdg-open: links: not found
    /usr/bin/xdg-open: 778: /usr/bin/xdg-open: lynx: not found
    /usr/bin/xdg-open: 778: /usr/bin/xdg-open: w3m: not found
    xdg-open: no method available for opening 'https://accounts.google.com/o/oauth2/auth?client_id=603366585132-0l3n5tr582q443rnomebdeeo0156b2bc.apps.googleusercontent.com&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fbigquery%20https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcloud-platform%20https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.email&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&response_type=code'

1. Here's where it gets tricky.  It'll look like it's only giving a list of errors. But if you look closely the error contains a url you see an https://accounts.google.com url in their.  Copy/paste that url into a new window and you'll see the usual Google Auth page.

   ![](/media/2019-08-05-b2.png)

Log in and give Tidyverse the permissions it's requesting.  It'll take you to a screen giving you a secret code:

![](/media/2019-08-05-b3.png)

Copy that code and paste  it into your JupyterLab terminal and hit Enter.

I know, it doesn't look like the terminal is waiting for any kind of input, but it actually is (hopefully gargle will fix this soon).

Sample output:

    > library(httpuv)
    > library(gargle)
    > library(bigrquery)
    > bq_auth(use_oob = TRUE)
    Is it OK to cache OAuth access credentials in the folder '/home/jupyter/.R/gargle/gargle-oauth' between R sessions?
    
    1: Yes
    2: No
    
    Selection: 1
    Enter authorization code: /usr/bin/xdg-open: 778: /usr/bin/xdg-open: www-browser: not found
    /usr/bin/xdg-open: 778: /usr/bin/xdg-open: links2: not found
    /usr/bin/xdg-open: 778: /usr/bin/xdg-open: elinks: not found
    /usr/bin/xdg-open: 778: /usr/bin/xdg-open: links: not found
    /usr/bin/xdg-open: 778: /usr/bin/xdg-open: lynx: not found
    /usr/bin/xdg-open: 778: /usr/bin/xdg-open: w3m: not found
    xdg-open: no method available for opening 'https://accounts.google.com/o/oauth2/auth?client_id=603366585132-0l3n5tr582q443rnomebdeeo0156b2bc.apps.googleusercontent.com&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fbigquery%20https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcloud-platform%20https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.email&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&response_type=code'
    4/lgjskDFGSjkwETSgsjGSKEJTssfgKWlgjskDFGSjkwETSgsjGSKEJTssfgKWlgjsk <===== the GCP auth code I copy/pasted in
    >

Now you can verify that your credentials have actually been cached.

    > bq_auth(use_oob = TRUE) <===== retrying the auth to see if it worked
    The bigrquery package is requesting access to your Google account. Select a pre-authorised account or enter '0' to obtaina new token. Press Esc/Ctrl + C to abort.
    
    1: xxxxxxx@gmail.com  <===== The auth credentials were cached
    
    Selection: 1

If you now try to authenticate to bigrquery using your email, it'll work (if bq_auth() returns with no message then that means it worked. Not the most intuitive, I know)

    > bq_auth(email="xxxxxxx@gmail.com")
    >

Now you can create a new R notebook within Jupyter Lab and authenticate yourself!

Create a new R notebook:

![](/media/2019-08-05-b4.png)

Run the following code within your notebook. It'll pull the authentication credentials for the given email addresses from the cache saved earlier:

    library(httpuv)
    library(gargle)
    library(bigrquery)
    
    bq_auth(email="xxxxxxx@gmail.com")
    
    project_id <- 'my-project-id'
    test_query_text <- "SELECT * FROM `bigquery-public-data.usa_names.usa_1910_current` LIMIT 10"
    
    test_results <- query_exec(test_query_text, project_id, use_legacy_sql = FALSE)
    
    test_results # print the results

# Option 2: Authenticate using a Service Account

This method involves creating a new service account in GCP, saving the key for that service account on your notebook, and using that key to authenticate to GCP.

Full instructions for using this method are available here:

[https://cloud.google.com/ml-engine/docs/notebooks/use-r-bigquery#create_a_service_account_key](https://cloud.google.com/ml-engine/docs/notebooks/use-r-bigquery#create_a_service_account_key "https://cloud.google.com/ml-engine/docs/notebooks/use-r-bigquery#create_a_service_account_key")