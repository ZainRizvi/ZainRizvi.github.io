---
layout: post
title: How to Use GPUs with R in Jupyter Lab
excerpt: Setting up R to use GPUs can be a pain, but it doesn't have to be.  Have
  you ever tried installing drivers for your Nvidia GPU?  The first time I tried it
  I spent the better half of an afternoon trying to get that done.  Though once I
  realized that I had to also recompile a lot of the software I wanted to actually
  use those GPUs, I was ready to give up.
tags:
- Install R on Jupyter Lab
- Google Cloud Platform
- AI Platform Notebooks
- Jupyter lab
- R language
- Jupyter Notebooks

---
Have you ever tried installing drivers for your Nvidia GPUs?  The first time I tried, I spent the better half of an afternoon trying to get that done.

And once I realized I also had to recompile multiple packages to actually use those GPUs, I was one error message away from being this guy:

![Frustration can lead to unfortunate outcomes](/media/2019-11-26-frustration.jpg "I was one error message away from being this guy")

Things have gotten a lot better since then.

In this post I'll share an easy way to setup your R language Jupyter Notebooks to use GPUs.  (Though if you prefer to use R outside of a notebook, these steps let you do that too)

It's a deep dive into one slide of [a talk I gave](https://youtu.be/FZvdaZ5jpXA) at Nvidia's GTC 2019 conference a few weeks ago.

# The Easy Way

There are three things you need to get going:

1. A machine with Nvidia GPU drivers installed
2. Install R and Jupyter Lab
3. Compile those R packages which require it for use with GPUs

If you use [AI Platform Notebooks](https://cloud.google.com/ai-platform-notebooks/) or [Deep Learning VM images](https://cloud.google.com/deep-learning-vm/), the Nvidia GPU drivers will be pre-installed for you (notebooks will give you the easiest experience).  You can also find offerings form other companies which also have the drivers pre-installed, taking care of step 1.

Once your machine with GPU drivers is ready, SSH into it and run the following command:

    sudo -- sh -c 'wget -O - https://raw.githubusercontent.com/ZainRizvi/UseRWithGpus/master/install-r-gpu.sh | bash'

There, one line and you're done.

It downloads [a script from my GitHub repository](https://github.com/ZainRizvi/UseRWithGpus/blob/master/install-r-gpu.sh) and executes it on your machine, handling all the tricky parts. That's it, you can now stop reading this article.

However, if you're anything like me, you may be a _liiiiittle_ bit wary of running random code from the internet.

Lets go deeper into what exactly this script does and make sure it's safe to run.

# What's going on here?

I'll walk through the code step by step to explain what it does.  You can [open up the code on GitHub](https://github.com/ZainRizvi/UseRWithGpus/blob/master/install-r-gpu.sh) if you'd like to see the full file.

## 1. Install common packages

Let's take it from the top:

    #!/bin/bash
    
    # Install R
    
    #required by multiple popular R packages
    apt install -y \
        libssl-dev \
        libcurl4-openssl-dev \
        libxml2 \
        libxml2-dev

We're installing some packages via apt.  These are dependencies for some of the R packages we want.

Seem safe enough

## 2. Installing R

Turns out installing R is a little complicated.  You have to:

1. Install additional dependencies
2. Add a whole new repository to your config
3. Tell your computer to trust that new repository
4. Then install r, presumably from that new repository

And the code for it:

    # Install the lastest version of R from the offical repository
    apt install apt-transport-https software-properties-common ocl-icd-opencl-dev -y
    apt install dirmngr --install-recommends -y
    apt-key adv --keyserver keys.gnupg.net --recv-key 'E19F5F87128899B192B1A2C2AD5F960A256A04AF'
    
    add-apt-repository "deb http://cloud.r-project.org/bin/linux/debian stretch-cran35/"
    
    apt update
    apt install r-base -y

The steps start to seem a bit iffy here (add a new key? a new repository?), but these are indeed part of [the official instructions]().  Feels shady, but it really is legit. The official docs and various other tutorials all say the same.

(Still feels like 👇)

![It's perfectly safe, I assure you](/media/2019-11-27-safe.jpg "It's perfectly safe, I assure you")

## 3. Integrate with Jupyter Lab/Jupyter Notebooks

Now we setup Jupyter Lab (or Jupyter Notebooks if you're using that) to use R.

We install the IRkernel and register it with Juptyer.

You can skip this step if you're not planning to use Jupyter Lab or Jupyter Notebooks

    # Install IRkernel
    Rscript -e "install.packages(c('repr', 'IRdisplay', 'IRkernel'), type = 'source', repos='http://cran.us.r-project.org')"
    
    # Register IRkernel with Jupyter
    Rscript -e "IRkernel::installspec(user = FALSE)"

## 4. Install your favorite R packages

This part is nice an simple.  We install whatever R packages you want from [CRAN](https://cran.r-project.org/).  Feel free to install a different set of packages from what I chose.

Note that over here you can only install those packages which do _not_ need to be recompiled for usage with GPUs.  The notable example is [XGBoost](https://machinelearningmastery.com/gentle-introduction-xgboost-applied-machine-learning/) (a handy ML library), which I'm not installing here.  It'll need to be recompiled and I'll do that further down.

    # Install various R packages
    
    function install_r_package() {
        PACKAGE="${1}"
        echo "installing ${PACKAGE}"
        Rscript -e "install.packages(c('${PACKAGE}'))"
        # install.packages always returns 0 code, even if install actually failed
        echo "validating install of  ${PACKAGE}"
        Rscript -e "library('${PACKAGE}')"
        if [[ $? -ne 0 ]]; then
            echo "R package ${PACKAGE} failed to install."
            exit 1
        fi
    }
    
    function install_r_packages() {
        PACKAGES=(${@})
        for PACKAGE in "${PACKAGES[@]}"; do
            install_r_package "${PACKAGE}"
        done
    }
    
    # Install google specific packages
    CLOUD_PACKAGES=( \
      'cloudml' \
      'bigrquery' \
      'googleCloudStorageR' \
      'googleComputeEngineR' \
      'googleAuthR' \
      'googleAnalyticsR' \
      'keras' \
    )
    install_r_packages "${CLOUD_PACKAGES[@]}"
    
    # Install other packages
     OTHER_PACKAGES=( \
       'tidyverse' \
       'httpuv' \
       'ggplot2' \
       'devtools' \
       'gpuR' \
       'xgboost' \
    )
    
     install_r_packages "${OTHER_PACKAGES[@]}"

## 5. Setup the default installation dir for your R packages

By default R will write packages to a location which is not writeable without sudo access, making it tricky to install packages, especially from within a Jupyter notebook.

The below code sets up a new directory `~/.R/library` to be used as the default location.  This requires creating a default environment variable that will always be set on boot, and verifying that the folder always exists every time your VM boots up.

    # Setup the default location for user-installed packages
    export R_LIB_SETUP="/etc/profile.d/r_user_lib.sh"
    cat << 'EOF' > "$R_LIB_SETUP"
    export R_LIBS_USER=~/.R/library
    # Ensure this directory exists at startup.  It needs to be in a persistent,
    # user writable location.
    mkdir -p "${R_LIBS_USER}"
    EOF
    
    chmod +x "${R_LIB_SETUP}"

## 6. Compile and install XGBoost for GPU

This is the most complicated step of the whole process.

The default xgboost on CRAN doesn't support GPUs, so we have to compile it from scratch.

However, the version of cmake on Ubuntu is too out of date to be able to compile xgboost (at least that's the case on the default image used by AI Platform Notebooks).

A newer version is not available in the repository, so we have to download and install it directly.

    # Install cmake (required to compile xgboost)
    wget https://github.com/Kitware/CMake/releases/download/v3.16.0-rc2/cmake-3.16.0-rc2-Linux-x86_64.sh
    
    chmod +x cmake-3.16.0-rc2-Linux-x86_64.sh
    CMAKE_DIR=/opt/cmake-custom
    sudo mkdir $CMAKE_DIR
    sudo ./cmake-3.16.0-rc2-Linux-x86_64.sh --skip-license --prefix=$CMAKE_DIR --exclude-subdir
    rm cmake-3.16.0-rc2-Linux-x86_64.sh
    
    sudo ln -s $CMAKE_DIR/bin/* /usr/local/bin

The steps are:

1. Download the cmake v3.16.0 installer
2. Make it executable
3. Create a directory for it to install the script to
4. Execute the installer
5. Clean up afterwards
6. Add the new cmake to PATH

And then of course we have to compile xgboost itself:

    # Install xgboost
    cd
    git clone --recursive https://github.com/dmlc/xgboost
    cd xgboost
    mkdir build
    cd build
    cmake -DUSE_CUDA=ON -DR_LIB=ON -DR_LIB=ON -DUSE_NCCL=ON -DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-10.1 -DNCCL_ROOT=/usr/local/nccl2 ..
    
    sudo make -j4
    sudo make install

That's another download build and install.

Note that the cmake command takes a bunch of flags.  The current command is optimized for running on AI Platform Notebooks, but you'll want to modify `--DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-10.1` to instead point to whatever location your own CUDA files are located

## 7. Install rpy2 for Python + R magic

Ok, this step isn't strictly necessary, but it lets you do something really cool.  With this you'll be able to create Python notebooks and then call R functions from _inside the Python notebook!!!_

You can even pass variables back and forth between the two languages!  You can run python code, get an output table, pass that output to your R code to view the data in a pretty graph.

It'll let you use each language for whatever it's best at, using the best tool for each job!

    function install_pip2_package() {
        pip2 install --upgrade --upgrade-strategy only-if-needed --force-reinstall "$@" || exit 1
    }
    
    function install_pip3_package() {
        pip3 install --upgrade --upgrade-strategy only-if-needed --force-reinstall "$@" || exit 1
    }
    
    function install_pip_package() {
        install_pip2_package "$1"
        install_pip3_package "$1"
    }
    
    # Install rpy2
    
    # To invoke R code in a python notebook, run the following code in a cell:
    #   import rpy2.robjects as robjects
    #   import rpy2.robjects.lib.ggplot2 as ggplot2
    #   %load_ext rpy2.ipython
    #
    # Then you can use the %R and %%R magic commands to run R code
    
    install_pip_package tzlocal # required by rpy2
    # 3.0.5 is the last version that works with Python 3.5
    install_pip3_package rpy2==3.0.5 # Code in both Python & R at the same time

## 8. Restart your VM

Remember how in step 5 we created a file to set your environment variable at boot time?  We never actually executed that file.

Lets reboot your machine now so that script takes effect

    # Reboot so that R user-installed packages path change takes effect
    sudo reboot

# Aaaaaaaand Done

Whew, that was a lot of steps.  It would be a pain to run those every time you create a new VM.  Fortunately you can just download and run [the script](https://github.com/ZainRizvi/UseRWithGpus/edit/master/install-r-gpu.sh) I mentioned earlier, and directly start using GPUs within your R notebooks.

# Want to make it even Faster?

The above script is convenient, but it still takes a good amount of time for it to finish running (around X0 minutes).  Personally, I'd rather not wait that long for my notebook to be ready.

If you'd like to have your notebook be ready in just two minutes instead of twenty, you can create a [Custom Deep Learning container](https://cloud.google.com/blog/products/ai-machine-learning/introducing-deep-learning-containers-consistent-and-portable-environments) with all of the above pre-installed.

**I'll be writing instructions on how to set up Custom Deep Learning Containers for GPU-based R projects (coming soon). Subscribe below to get an email when the article is ready!**