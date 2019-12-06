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

Now cd to the directory that contains that file and run `docker build .`  And Docker will download that image from the GCP repository, apply your custom label to it, and save the resulting image locally.

_Note: If you name your dockerfile anything other than "Dockerfile" then you'll need to also specify the file name via a_ `_-f \[filename\]_` _parameter, so your command would look like_ `_docker build . -f \[filename\]_`

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

Let's extend this Dockerfile to add R support to the Tensorflow notebook.

I've already shared a script on GitHub which can install R onto your AI Platform Notebooks, but that script takes way too long to run to invoke it on every notebook you create.  Let's run it just once when creating the container and use that container for our future notebooks instead.

(You can read more about what the script does **here**)

We'll add two lines to our container to run that script:

    FROM gcr.io/deeplearning-platform-release/tf2-cpu
    LABEL maintainer="Zain  Rizvi"
    
    RUN apt update -y 
    # Add R support
    RUN wget -O - https://raw.githubusercontent.com/ZainRizvi/UseRWithGpus/master/install-r-cpu-ubuntu.sh | bash

And now we can run `docker build .` again.  This time the command will take a **long** time to complete (that script takes a little under an hour to run, and we're no longer running inside the fast GCP network which would at least improve our download speeds).

But once completes, we'll again be given a new image Id similar to the one we saw earlier.  Just tag that and push it to your registry the same way we did before

xxxx instructions xxx

And now your registry is available to use

# When things go wrong

What do you do if you get errors in the middle of your Docker build?  If the console logs don't tell you exactly what's wrong, you can open up your docker image and execute the relevant instructions manually to debug the situation.  That way when you hit the error you can explore the environment and see what it takes to make that error go away.  

Run the following command to get an interactive terminal:

`docker run -it [image_name] /bin/bash`

For example:

`docker run -it gcr.io/deeplearning-platform-release/tf2-cpu /bin/bash`

The downside to this is that you have to execute all the docker steps up till that point, and if your build  process takes a long time (like it does with the script I'm using) that could be a lot of time lost just waiting for a build to run.

There are ways you can reduce the time you spend waiting though.  It depend on how Docker creates images in layers.  Basically, pretty much every line in a Dockerfile results in a new layer being created, and you can see that layer in both the output of your build command and in the output when you run `docker images`

    $ docker build . 
    Sending build context to Docker daemon  2.048kB
    Step 1/4 : FROM gcr.io/deeplearning-platform-release/tf2-cpu
     ---> e493f17c90d0
    Step 2/4 : LABEL maintainer="Zain  Rizvi"
     ---> Using cache
     ---> 5ae08c1b46d1
    Step 3/4 : RUN apt update -y
     ---> Using cache
     ---> c189b73c08b5

When it can, Docker likes to reuse layers from previous runs in order cut down on build times. In the above command you can see that Step 2 and Step 3 are using previously created images from the cache.  

I'm not sure how it decides if it's safe to re-use an old layer, but if we split our installation script into multiple parts and have each part be a separate RUN line in the Dockerfile, it'll cut down on the time it takes to test (and rebuild) our images.

If we split some long, time-intensive step this way then instead of calling `docker run` on the base image we used, we could instead call it on one of the Image Ids from our intermediate stages.

The keen-eyed among you may have noticed that the script I'm running in the above example is install-r-cpu-ubuntu.sh instead of the install-r-cpu.sh script I mentioned in my [original post](https://zainrizvi.io/blog/using-gpus-with-r-in-jupyter-lab/).  Do you remember that there was a step where we added a new key and a new repository?

> The steps start to seem a bit iffy here (add a new key? a new repository?), but these are indeed part of [the official instructions](https://zainrizvi.io/blog/using-gpus-with-r-in-jupyter-lab/). Feels shady, but it really is legit. The official docs and various other tutorials all say the same.
>
> (Still feels like 👇)
>
> ![It's perfectly safe, I assure you](https://zainrizvi.io/media/2019-11-27-safe.jpg "It's perfectly safe, I assure you")

So it turns out every distro (and every version of each distro) needs it's own repository and it's own key.  And while AI Platform Notebook images run on a Debian OS, the Deep Learning Container Images are built with an Ubuntu base, so we have to point to the Ubuntu repository instead (twenty minutes spent debugging and testing that problem... ಠ_ಠ)