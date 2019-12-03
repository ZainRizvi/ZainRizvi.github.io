---
layout: post
title: Using Custom Deep Learning Containers
excerpt: ''
tags: []

---
1. Value Prop
2. What are custom containers?
3. Prequisites
4. Which ones are available
5. How do I edit them
6. Example using R on Tensorflow

Ever find yourself needing to install the same packages on all your deep learning notebooks?  Or wishing you could send your exact setup to someone else who could run your notebook? Or maybe you're a corporation which wants all your data scientists to have some internal libraries on all their notebooks.

GCP's AI Platform Notebooks team offers Deep Learning Containers, which is a containerized version of the exact same images you get when you create a regular AI Platform Notebook (full disclosure: that's my team).

And those containers are 100% free

A quick list of benefits you can expect by using these:

* Ability to run these deep learning environments anywhere, including directly on your laptop
* Avoid having to customize your notebook environment every time you create a new notebook
* Have a consistent environment used by all of your data scientists
* Ability to modify/replace default Jupyter Lab environment (if you want to)

# Prerequisites

In order to follow along with the rest of the post I'll assume you have the following installed on your computer:

docker

gcloud (optional)

# Download a container

We can take a quick look at what containers we have available to us by running

    gcloud container images list --repository="gcr.io/deeplearning-platform-release"

Currently that command outputs:

    > gcloud container images list --repository="gcr.io/deeplearning-platform-release"
    NAME
    gcr.io/deeplearning-platform-release/base-cpu
    gcr.io/deeplearning-platform-release/base-cu100
    gcr.io/deeplearning-platform-release/beam-notebooks
    gcr.io/deeplearning-platform-release/pytorch-cpu
    gcr.io/deeplearning-platform-release/pytorch-cpu.1-0
    gcr.io/deeplearning-platform-release/pytorch-cpu.1-1
    gcr.io/deeplearning-platform-release/pytorch-cpu.1-2
    gcr.io/deeplearning-platform-release/pytorch-cpu.1-3
    gcr.io/deeplearning-platform-release/pytorch-gpu
    gcr.io/deeplearning-platform-release/pytorch-gpu.1-0
    gcr.io/deeplearning-platform-release/pytorch-gpu.1-1
    gcr.io/deeplearning-platform-release/pytorch-gpu.1-2
    gcr.io/deeplearning-platform-release/pytorch-gpu.1-3
    gcr.io/deeplearning-platform-release/r-cpu
    gcr.io/deeplearning-platform-release/r-cpu.3-6
    gcr.io/deeplearning-platform-release/tf-cpu
    gcr.io/deeplearning-platform-release/tf-cpu.1-13
    gcr.io/deeplearning-platform-release/tf-cpu.1-14
    gcr.io/deeplearning-platform-release/tf-cpu.1-15
    gcr.io/deeplearning-platform-release/tf-gpu
    gcr.io/deeplearning-platform-release/tf-gpu.1-13
    gcr.io/deeplearning-platform-release/tf-gpu.1-14
    gcr.io/deeplearning-platform-release/tf-gpu.1-15
    gcr.io/deeplearning-platform-release/tf2-cpu
    gcr.io/deeplearning-platform-release/tf2-cpu.2-0
    gcr.io/deeplearning-platform-release/tf2-gpu
    gcr.io/deeplearning-platform-release/tf2-gpu.2-0
    
That's a list of all the different environments available for you to choose from.  You can see Tensorflow, Pytorch, R, and others on the list, and most of them come in both CPU and GPU variations.

