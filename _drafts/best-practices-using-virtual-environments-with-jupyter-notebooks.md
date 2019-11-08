---
layout: post
title: "[Best Practices] Using Virtual Environments with Jupyter Notebooks"
excerpt: Bring the best practices established by the Python community to your Jupyter
  Notebooks.  You can add virtual environments to Jupyter Lab, giving each notebook
  it's own environment.  This post goes into detail explaining exactly how you can
  add virtual environments to your own notebooks on Google Cloud's AI Platform Notebooks
tags:
- Jupyter Lab
- technical

---
Using Virtual Environments has become a standard best practice in the Python community.  They allow you to work on multiple python projects at the same time, without one accidentally corrupting the dependencies of another.  While using these Virtual Environments has become the norm with Python projects, they haven't yet caught on in Python notebooks.  However, they're easy to add to your Jupyter Notebook or Jupyter Lab setup.  This post will describe just how you can use them

# What will this enable you to do?

By following these steps, you'll be able to have multiple notebooks running on the same machine in Jupyter Lab where each notebook uses own versions of potentially conflicting python packages.

You'll do this by creating multiple python virtual environments (which are isolated from each other) and having different notebooks run inside their own environment.

# Why does it matter?

**Benefit from established Python best practices!**  Having each project in a separate virtual environment is an existing best practice for python projects, so it seems logical to extend this behavior to python notebooks as well.   Of course, this will only apply to Python notebooks and not notebooks using other languages

# How do you set it up?

First you'll need a Jupyter Lab notebook environment.  If you don't have one already you can quickly create one us Google Cloud's [AI Platform Notebooks](https://cloud.google.com/ai-platform-notebooks/)

Go to your Jupyter Lab notebook and in it's terminal enter the following (replacing "myenv" with whatever name you want to give your environment):

    # You only need to run this command once per-VM
    sudo apt-get install python3-venv 
    
    # The rest of theses steps should be run every time you create
    #  a new virtual environment
    
    # The '--system-site-packages' flag allows the python packages 
    #  we installed by default to remain accessible in the virtual 
    #  environment.  It's best to use this flag if you're doing this
    #  on AI Platform Notebooks so that you keep all the pre-baked 
    #  goodness
    python3 -m venv myenv --system-site-packages
    source myenv/bin/activate #activate the virtual env
    
    # Register this env with jupyter lab. It’ll now show up in the
    #  launcher & kernels list once you refresh the page
    python -m ipykernel install --user --name=myenv
    
    # Any python packages you pip install now will persist only in
    #  this environment_
    deactivate # exit the virtual env

Launcher after I added two customized environments (“myenv” and “zainsenv”)