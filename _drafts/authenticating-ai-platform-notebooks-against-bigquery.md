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

The below instructions describe how to 