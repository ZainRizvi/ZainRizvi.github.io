---
layout: post
title: Authenticating to BigRQuery on GCP AI Platform Notebooks (or any remote Jupyter
  Lab instance)
excerpt: When you're using Jupyter Notebooks with R, authenticating with BigQuery
  can be tricky.  Here's how you can do it
tags:
- BigRQuery
- BigQuery
- Jupyter Lab
- R
- Google Cloud Platform

---
GCP AI Platform Notebooks makes it really easy to run notebooks on Jupyter Lab and even offers R language notebooks.  R is great for crunching large data sets, and a popular place for people to store their data is on BigQuery.

The most popular library for accessing BigQuery in R is the open source library [BigRQuery](https://github.com/r-dbi/bigrquery).  It's an extremely useful library, but it has the downside that the authentication step will try one of two things:

1. It either will prompt people for extra input on the command line
2. Or open up a port on http://localhost to listen for GCP authentication

\#1 doesn't work with any Jupter  Notebook since they are not designed to accept extra commands in the middle of an execution

\#2 won't work if you're connected to Jupyter Lab on a remote machine (the http://localhost will point you to the wrong VM!)

Since neither of the normal ways to authenticate yourself will work, this post describes two different methods you can use to authenticate yourself to BigQuery within a AI Platform Notebook:

1. Authenticate using your normal GCP user credentials
2. Authenticate using a service account

# Prereq: Create an AI Platform Notebook for R

1. Go to [http://console.cloud.google.com/mlengine/notebooks/instances](http://console.cloud.google.com/mlengine/notebooks/instances "http://console.cloud.google.com/mlengine/notebooks/instances")
2. Select 'New Instance' -> 'R 3.x' -> Create
3. Wait for the notebook to be created and click "OPEN JUPYTERLAB"

# Option #1: Authenticating using your GCP user credentials

This method uses the Jupter Lab terminal to run the interactive commands and cache the credentials. Once you're authenticated, you can switch to a notebook and it'll use the credentials in the cache.

1. In Jupyer Lab open a new terminal (open a new launcher and click 'Terminal')

   ![](https://screenshot.googleplex.com/qhRWn3FT0ZY.png)
2. Start R

    jupyter@r-20190802-172922:~$ R
    
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
    
    >

3. Install the required packages

As of this writing, BigRQuery needs the dev version of gargle for this authentication to work.  Later you shouldn't need to explicitly install gargle.

    install.packages("httpuv")
    install.packages("devtools") 
    devtools::install_github("r-lib/gargle") # get the dev version of gargle
    install.packages("bigrquery") 
    install.packages("readr") # To read BigQuery results

The above packages will take \~10 minutes to install

4. Import the required libraries
5. Authenticate yourself by running the command `bq``_auth(use_``oob = TRUE)` (correct your email address as appropriate). 

    library(httpuv)
    library(gargle)
    library(bigrquery)
    bq_auth(use_oob = TRUE)

6. Say yes when it asks about caching the OAuth credentials. You'll see an error like the following

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
    

7. Here's where it gets tricky.  It'll look like it's only giving a list of errors. But if you look closely the error contains a url yo