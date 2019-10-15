---
layout: post
title: Authenticating AI Platform Notebooks against BigQuery
excerpt: How to authenticate with BigQuery (in Python) when using AI Platform Notebooks
tags:
- GCP
- AI Platform Notebooks
- BigQuery

---
When you use [AI Platform Notebooks](https://cloud.google.com/ai-platform-notebooks/) by default any API calls you make to GCP use the default compute service account that your notebook runs under.  This makes it easy to start getting stuff done, but sometimes you may want to use BigQuery to query data that your service account doesn't have access to.

The below instructions describe how to use your personal account to authenticate with BigQuery.  This specifically applies to authentication when using a python based notebook.  If you want to authenticate on a R based notebook you can find [instructions for that here](https://www.zainrizvi.io/blog/authenticating-to-bigrquery-on-gcp-ai-platform-notebooks/).

Normally you would use `gcloud auth login` from the jupyer lab terminal to login to your personal user account and call Google apis, but the BigQuery library auth works differently for some reason.

Instead, you need to create a credential object containing your user credentials and pass that to the bigquery library.

1. Install the `pydata_google_auth` package:

   `%pip install pydata_goog_auth`
2. Restart the kernel: Kernel -> Resart Kernel

   ![](https://screenshot.googleplex.com/SXzOG3pCaBk.png)
3. Import the library and create your credentials:

       import pydata_google_auth
       credentials = pydata_google_auth.get_user_credentials(
           ['https://www.googleapis.com/auth/bigquery'],
       )
4. When you execute the above cell you'll see an output with an authentication link and a text box

   ![](https://screenshot.googleplex.com/KJ13JmkmkLd.png) 
5. Copy that link, paste it into a browser, and authenticate with google.  You'll see an authorization code similar to the below:

   ![](https://screenshot.googleplex.com/1g35DesEv29.png)
6. Copy that code and paste it into the authentication code input box you saw in your notebook

   ![](https://screenshot.googleplex.com/v6cAGhKSn3S.png)
7. 