---
layout: post
title:  "Installing NVIDIA drivers for CUDA on Optimus/Ubuntu"
date:   2017-04-03 18:20:19
category: Tips & Hacks
---

# The Problem

[Optimus](https://en.wikipedia.org/wiki/Nvidia_Optimus) enabled machines allow switching between the Intel display interface and a secondary NVIDIA graphics card. This is usually a technology on Laptops, particularly Lenovo machines. Using the NVIDIA card, with it's higher power usage, is usually overkill for displaying a standard Ubuntu OS GUI. This is just as well because installing the NVIDIA drivers on Ubuntu/Optimus has a minefield of issues. Most times, the user will install the NVIDIA drivers only to find themselves confronted by the Black Screen of death on reboot, or an endless login loop. There are additional technologies which aim to ease the balancing of two display drivers, e.g. [Bumblebee](https://bumblebee-project.org/). However, this just adds to the complexity of the situation.

Ok, so we just stick with the Intel drivers right? But what if, like me, you want to make use of the NVIDIA GPU. For example, via the [CUDA APIs](https://developer.nvidia.com/cuda-zone). In these instances our task is even more tricky as we want to install the NVIDIA drivers, but not use them as a display driver.

# The Solution

There is a great blog post on [Hemens Notepad](https://hemenkapadia.github.io/blog/2016/11/11/Ubuntu-with-Nvidia-CUDA-Bumblebee.html) which probably gets most people pretty far along the right path.

Unfortunately, I find that any installation of NVIDIA via the Ubuntu Package Manager (i.e. via apt-get install) results in the login loop mentioned above. This includes installing it directly with ```sudo apt-get install nvidia-XXX```, or via installation as part of CUDA (e.g. ```sudo apt-get install cuda-8-0```).

The reason for the login loop can be found by looking in ```/var/log/Xorg.0.log``` after an unsuccessful login. You will likely see the following error (search for "EE") :

```
(EE) Failed to initialize GLX extension (Compatible NVIDIA X driver not found)
```

The [GLX extension](https://en.wikipedia.org/wiki/GLX) is an interface between the Linux X display system and the OpenGL graphics library. When NVIDIA is installed, it overrides the native X OpenGL libraries with the the NVIDIA OpenGL libraries. Unfortunately this mix of NVIDIA and Intel libraries cause the GLX extension to fail as above. I have tried (and failed) to unpick the NVIDIA OpenGL libraries once installed (if anyone has a solution please comment!).

So how do we install the NVIDIA drivers without adding the OpenGL Libraries? First we need to install the driver directly from the [NVIDIA website](http://www.nvidia.co.uk/Download/index.aspx?lang=en-uk). Doing this, we can provide the --no-opengl-files flag.

Although it is not recommended to install via the NVIDIA run command as the resulting package is not "managed" and maintained by the Ubuntu package manager, it is a compromise we can live with as we are not using the drivers to actually display anything.

To summarise, the solution is to download the driver installation software from NVIDIA, and run the following command :

```
sudo ./NVIDIA-Linux-x86_64-375.39.run --no-opengl-files
```

Note - it is very important to answer *NO* to running the XConfig utility at the end of the installation routine. This would set up NVIDIA as the display device. 

After installation is complete, reboot, and you should find that the drivers are installed and it is still possible to login to Ubunutu unity.
