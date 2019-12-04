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

We'll take the Tensorflow 2 CPU image and modify it to create our custom environment

# Steps

Create your image

We'll create a super simple image first.  We'll use the Tensorflow 2 CPU image as our base and not change anything other than adding our own name as the maintainer of the new image.

To do this, create a file named "Dockerfile" and give it the following contents:

    FROM gcr.io/deeplearning-platform-release/tf2-cpu
    LABEL maintainer="Zain Rizvi"

Now cd to the directory that contains that file and run "docker build ."  And Docker will download that image from the GCP repository, apply your custom label to it, and save the resulting image locally. (If you name your dockerfile anything other than "Dockerfile" then you'll need to also specify the file name via a "-f \[filename\]" parameter.)

You'll see something similar to the following

    > docker build .
    Sending build context to Docker daemon  2.048kB
    Step 1/2 : FROM gcr.io/deeplearning-platform-release/tf2-cpu
    latest: Pulling from deeplearning-platform-release/tf2-cpu
    35c102085707: Already exists
    251f5509d51d: Already exists
    8e829fe70a46: Already exists
    6001e1789921: Already exists
    1259902c87a2: Already exists
    83ca0edf82af: Already exists
    a459cc7a0819: Pull complete                                                                                              
    7de7778cb300: Pull complete                                                                                              
    62e0a31a8af6: Pull complete                                                                                              
    a7785d29f5ab: Pull complete                                                                                              
    6b76a06da4d7: Pull complete                                                                                              
    413905cedc93: Pull complete                                                                                              
    a5d245cced6f: Pull complete                                                                                              
    8c6be6aa5553: Pull complete                                                                                              
    1d7154118978: Pull complete                                                                                              
    1df8626a77b0: Pull complete                                                                                              
    307ee6add651: Pull complete                                                                                              
    5347a4f2b51b: Pull complete                                                                                              
    68eab8b2c13c: Pull complete                                                                                              
    928e12577c37: Pull complete                                                                                              
    48d9ceba06f1: Pull complete                                                                                              
    Digest: sha256:88ae24914e15f2df11a03486668e9051ca85b65f8577358e7d965ce6a146f217
    Status: Downloaded newer image for gcr.io/deeplearning-platform-release/tf2-cpu:latest
     ---> e493f17c90d0
    Step 2/2 : LABEL maintainer="Zain Rizvi"
     ---> Running in 561cbb80b0c5
    Removing intermediate container 561cbb80b0c5
     ---> 8cee7adcf9c3
    Successfully built 8cee7adcf9c3

Note the id in the last line `Successfully built 8cee7adcf9c3`. That `8cee7adcf9c3` is a local image id, and it will be important when we want to push our image (a couple steps down).

To push your image, you need a registry to push it to.  I'll give you instructions that put your registry on Docker Hub (which is free for public registries) but you can use whatever registry provider you prefer.  For a Docker Hub registry you can go to hub.docker.com and create your public registry (do that now).  You'll need to create an account first though if you don't have one already

Before the push, make sure you're logged into docker from within the console:

docker login --username zainrizvi --password-stdin

Now to push we need to tell docker which image it should be pushing to our new registry.  We do this by tagging the image we built with the path of our registry and add an optional tag (yeah, the overload of the word 'tag' is a bit annoying).

Remember that image Id I told you to note earlier (mine was `8cee7adcf9c3`), now is when you need that.  We'll tag that Id with the path to the repository we want to use:

    docker tag 8cee7adcf9c3 zainrizvi/deeplearning-container-generic:latest

If you run `docker images` you should now see an image with that repository and tag

    > docker images                                                                           
    REPOSITORY                                         TAG                 IMAGE ID            CREATED             SIZE      
    zainrizvi/deeplearning-container-generic           latest              8cee7adcf9c3        4 minutes ago       6.26GB   

However, just because we've tagged the image doesn't mean it actually exists in the repository.  We have to do a docker push to get it in there:

    > docker push zainrizvi/deeplearning-container-generic
    The push refers to repository [docker.io/zainrizvi/deeplearning-container-generic]
    3bc6581319d1: Pushed
    9460d92bb97d: Pushed
    11e8218cebaf: Pushed
    918c750480c5: Pushed
    802f1a04733b: Pushed
    07a867e0ba2d: Pushed
    092c50747c65: Pushed
    d6fb36f9bda1: Pushed
    f36c7efe6784: Pushed
    1f3727b0e386: Pushed
    80824689ea9a: Pushed
    dae8971c1728: Pushed
    63b763e1ea3e: Pushed
    7d412b9c88ab: Pushed
    4019db0181d2: Pushed
    5a78197acff6: Pushed
    804e87810c15: Pushed
    122be11ab4a2: Pushed
    7beb13bce073: Pushed
    f7eae43028b3: Pushed
    6cebf3abed5f: Pushed
    latest: digest: sha256:51a109b02534ec04875051cb36cc6ede6d50b9af136f0e2725a77500d7e17852 size: 4717

And now if you go to your docker registry you'll see that the image is there for anyone to view and download

So that was cool, but we didn't really do anything special. We're not pre-configuring any of the packages we really need or anything like that.

Let's now add some actual customizations to this image

# Customizing your image