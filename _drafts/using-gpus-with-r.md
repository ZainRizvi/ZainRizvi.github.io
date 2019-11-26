---
layout: post
title: Using GPUs with R
excerpt: Setting up R to use GPUs can be a pain, but it doesn't have to be.  Have
  you ever tried installing drivers for your Nvidia GPU?  The first time I tried it
  I spent the better half of an afternoon trying to get that done.  Though once I
  realized that I had to also recompile a lot of the software I wanted to actually
  use those GPUs, I was ready to give up.
tags:
- Google Cloud Platform
- AI Platform Notebooks
- Jupyter lab
- R language
- Jupyter Notebooks

---
Have you ever tried installing drivers for your Nvidia GPU?  The first time I tried it I spent the better half of an afternoon trying to get that done.  Though once I realized that I had to also recompile a lot of the software I wanted to actually use those GPUs, I was ready to give up.

Things have gotten much better now.

In this post I'll be sharing an easy way to setup your R language Jupyter Notebooks to use GPUs.  Though if you prefer to use R outside of a notebook, these instructions will let you do that too (there's just one step you'll want to skip).

These instructions are a deep dive into one slide of [a talk I gave](https://youtu.be/FZvdaZ5jpXA) at Nvidia's GTC 2019 conference a few weeks ago.

# How do I do it?