---
layout: post
title:  "Turn a dedicated Kodi box into a Home Server using Docker"
date:   2017-02-28
category: Home Automation
---

<meta name="twitter:card" content="summary" />
<meta name="twitter:site" content="@cwilko" />
<meta name="twitter:title" content="Turn a dedicated Kodi box into a Home Server using Docker" />
<meta name="twitter:description" content="Wouldn't it be great if we didn't have to tie up a whole Raspberry Pi with a dedicated Kodi installation, but also have the ability to run some additional services. In other words, can we turn our Kodi box into a Home Server as well?" />
<meta name="twitter:image" content="http://www.pc-magazin.de/bilder/447505/500x300-c2/Kodi-Logo.jpg" />

# The Problem 

Running [OpenELEC](http://openelec.tv/)/[LibreELEC](https://libreelec.tv/) on a Raspberry Pi is a great way to get a minimum footprint [Kodi](https://kodi.tv/) box, with all the necessary OS performance tuning ready out of the box. However, wouldn't it be great if we didn't have to tie up a whole Raspberry Pi in this way, but also have the ability to run some additional services. In other words, can we turn our Kodi box into a Home Server as well?

For example, maybe we would like to also add a [MySQL server](https://dev.mysql.com/downloads/mysql/) to the box, for [hosting kodi config](http://kodi.wiki/view/MySQL) on the network. Usually this would involve installing MySQL onto the box and setting up the required database configuration. Unfortunately, the problem with running a dedicated Kodi OS is that we don't have access to tools like a package manager, which allow us to install additional software. Only the bare essentials needed to run Kodi are provided.

# The Solution

The solution presents itself with the addition of the [Docker add-on](https://github.com/lrusak/lrusak-openelec-addons) to the OpenELEC/LibreELEC repository!

[Docker](https://www.docker.com/) provides an application container framework. This means that once the Docker add-on is installed, we can spin up new applications which Docker hosts in self-contained and isolated containers. We don't need to care how Docker does it, but essentially every application thinks it is in it's own isolated OS and filesystem. The best bit, is that you don't actually need to manually install anything new into a container - because there is a whole [community](https://hub.docker.com/) of off-the-shelf applications (or "images") just waiting to be installed. In our example above involving MySQL, we don't need to install MySQL, we just need to point docker at a repository where some other clever sausage has create a MySQL image. Docker then fetches the application image, and loads this up in a new container. Voila - MySQL is running on our box.

You can think of each docker container as hosting a service on your Home Server. Spinning up new services and destroying old ones becomes as simple as running a command, or can even be done via one of a number of Docker dashboards.

On my Kodi LibreELEC box, I have docker containers which host three additional services for running MySQL, NodeRED, and an Amazon Alexa custom service. The first two of these are completely off-the-shelf, the only parameters provided are the ports to host the service, and a folder (or "volume") where files are shared between the main OS and the container.

Docker provides an incredibly powerful way of managing complex applications and services, in a simple and platform agnostic way.

Ok so that was the marketing pitch. The devil is in the details. Below I`ll show you how to set up Docker and create a MySQL server on your OpenELEC/LibreELEC linux box (e.g. RaspberryPi). This should give enough understanding to enable you to pluck other docker services which you may wish to host on your Kodi Server.

# Setting up Docker on OpenELEC/LibreELEC

Either OpenELEC or LibreELEC can be installed via the [NOOBS](https://www.raspberrypi.org/downloads/noobs/) loader. Alternatively it can be flashed from the relative locations. Once installed, the Pi will boot directly into the Kodi Media Centre interface. There are then two steps we need to perform to begin adding additional services. The first is to enable SSH on the box. This allows us to remotely access the Pi via another computer on the network. The second is to install the docker addon. 

**To enable SSH (on LibreELEC):**

- From the Kodi Confluence main menu, navigate to ```SYSTEM -> LibreELEC -> Services -> Enable SSH```

**To install the Docker add-on :**

- From the Kodi Confluence main menu, navigate to ```SYSTEM -> Add-ons -> Install from repository -> LibreELEC Add-ons -> Services -> Docker```

We should now be able to log in to the Raspberry Pi via an SSH session on another machine on the local network. From this remote machine, enter the following :

```bash
	$ ssh root@<IP address of the Pi> 

	<Provide the default password of "libreelec" or "openelec">
```

The last useful thing we need to do to in order to set up Docker is to have the Docker daemon listen on a particular port. By default it will listen on a local socket, which creates problems later on when we want to map ports to our containers, or when we want to access the docker daemon from a remote machine (e.g. to link a docker dashboard). To do this, edit the ```/storage/.kodi/addons/service.system.docker/system.d/service.system.docker.service``` file and add ```-H tcp://0.0.0.0:2375``` to the docker daemon call.

Your file should then contain something like the following :

```
	ExecStart=/storage/.kodi/addons/service.system.docker/bin/dockerd -H tcp://0.0.0.0:2375 --exec-opt native.cgroupdriver=systemd \
                                                                  --log-driver=journald \
                                                                  --group=root \
                                                                  $DOCKER_DAEMON_OPTS \
                                                                  $DOCKER_STORAGE_OPTS

```
After this you should reboot.

You can test everything is working by SSHing into the Raspberry Pi and entering the following to install a helloworld docker image :

```bash
	$ docker -H 0.0.0.0:2375 run hello-world

	Hello from Docker!
	This message shows that your installation appears to be working correctly.

	To generate this message, Docker took the following steps:
	...(snipped)...
```

# Installing MySQL (or any other application image) on a RaspberryPi

Docker images are constructed from *layers*. For example, the MySQL image may consist of three layers. A *base* image designed to interface with ARM technology (i.e. the Pi chipset), a second image to add a vanilla basic linux OS, and a third image which actually contains the installation of MySQL. When we ask docker to install a MySQL image, it will fetch all three to the local filesystem and create a single container which hosts the combined final image. These images are retrieved from the [Docker Hub repository](https://hub.docker.com/). This is a huge community-driven set of images, available for pretty much any architecture and application you can think of. If it doesn't exist, it is fairly straightforward to find something close which you could then tweak and contribute back to Docker Hub. For me, the off-the-shelf nature appeals. I don't really want to maintain my own custom images if I can help it.

Selecting the image for our Raspberry Pi Kodi Server takes a little thought. We are limited by the chipset that our device is running on, and also by the amount of memory available on the Pi.

Above I mentioned the ARM architecture which underpins the RaspberryPi. When we install images, we need to ensure they are built from a base image that is suitable for our device. The name of the image usually gives us a clue when looking for ARM images, for example... *armhf-<image name>*

The second image in our example above is the "OS image". Usually we want to choose an image that is based on an OS which matches our device - as docker is able to share resources. However, in this instance we have a Kodi dedicated OS on our device so we need to incorporate a layer that provides additional tools to support the MySQL application. We want this to be "just enough" to run MySQL. One such lightweight OS is [Alpine](https://www.alpinelinux.org/about/). Alpine provides a tiny footprint OS with a basic set of tools. A number of applications are available for install on Alpine, including MySQL.

Armed with the information above, we can set about finding an image on Docker Hub that provides MySQL, running on Alpine, based on ARM 

Here are a few examples : 

1. [https://hub.docker.com/r/tarzan79/alpine-mysql](https://hub.docker.com/r/tarzan79/alpine-mysql)
2. [https://hub.docker.com/r/wangxian/alpine-mysql/](https://hub.docker.com/r/wangxian/alpine-mysql)
3. [**https://hub.docker.com/r/abrahammouse/armhf-alpine-mysql/**](https://hub.docker.com/r/abrahammouse/armhf-alpine-mysql)

You can see there are many options to choose from. Often each image is subtley different, perhaps in the way the image is configured, or the directory structure it uses, or the amount of resources it uses, etc. The best way to find an image that suits your needs is to install a few and make a few comparisons. For a LibreELEC install, we have very limited memory remaining beyond the native Kodi install. After trialling a few images, I found that the last image above (abrahammouse/armhf) had the lowest memory footprint, and was therefore the correct choice.

When we install a docker image to a new container, we can "mount a volume", that is we can identify a folder on our local filesystem which will replace a folder on the underlying container image. In this way the image can access our OS filesystem, and vice versa. This is useful for specifying a config directory which we can use to configure the application within the image. It also means that when we destroy/recreate the docker container, the folder will remain and the files will be picked up by the new container. For example, we clearly don't want to lose the mysql db files when we destroy a container, so we therefore mount a volume which overrides the image db storage folder. 

We are therefore ready to create a docker container :

```bash
	$ docker run -H tcp://0.0.0.0:2375 --name=mysql --rm --volume=/storage/mysql:/etc/mysql \
	          --volume=/storage/mysql/data:/app:rw --volume=/storage/mysql/run:/run:rw \
	          --publish=3306:3306 \
	          --env=MYSQL_ROOT_PASSWORD=<KODI_ROOT_PW> --env=MYSQL_USER=kodi \
	          --env=MYSQL_PASSWORD=kodi \
	          abrahammouse/armhf-alpine-mysql
```
Let's examine this in more detail...

We call docker with the ```run``` commmand, which tells docker to fetch and run a docker image in a new container, we also supply the local port on which docker is listening. If the image has previously been fetched then the image will be loaded from local storage (and will therefore be subsequently very quick to start). We provide ```--name``` to specify an easy to reference name, ```--rm``` will ensure the container is removed when stopped. Next we mount some volumes, a folder is created on LibreELEC filesystem called "mysql". In here we create and map a few new folders to locations on the underlying image. This means that rather than use the image folders, docker will instead use the LibreELEC folders, and we will have visibility and persistence of the key config files and db data. Next we map the container MySQL server port to a real port on the Pi. Finally we provide some environment variables to MySQL which allow set up of some basic credentials (e.g. for accessing MySQL server via a client) - be sure to set an appropriate root password.

Once we have run this we should have a MySQL server running in the background, while Kodi is still available on the display.

If you take a look in the ```/storage/mysql``` folder you will see that a bunch of MySQL files have been created in our folder. This is where any database data will be stored. We can test the installation was successful by accessing the MySQL server from a remote client. From your client machine, type the following to see a summary of the db tables:

```bash
	$ mysql -uroot -p<KODI_ROOT_PW> -h <IP Address of the Raspberry Pi> -P 3306 -e 'SHOW VARIABLES WHERE Variable_Name LIKE "%dir"'
```
Provide the root pw created above.

Great! We have a running MySQL service which we can use for whatever we like (Such as [shared thumbnails/config across multiple kodi installations](http://kodi.wiki/view/MySQL)). However, we don't want to run the above command every time we restart our Kodi box. Instead, it's a good idea to have this docker container automatically start after a reboot. To do this we create a new file in the ```/storage/.config/system.d``` folder called ````mysql.service```` which contains the following :

```
	[Unit]
	Description=Mysql Container
	Requires=service.system.docker.service
	After=service.system.docker.service
	Before=kodi.service

	[Service]
	Restart=always
	RestartSec=10s
	TimeoutStartSec=0
	ExecStartPre=-/bin/sh -c "mkdir -p /storage/mysql"
	ExecStart=/storage/.kodi/addons/service.system.docker/bin/docker -H tcp://0.0.0.0:2375 run \
	          \
	          --name=mysql --rm --volume=/storage/mysql:/etc/mysql \
	          --volume=/storage/mysql/data:/app:rw --volume=/storage/mysql/run:/run:rw \
	          --publish=3306:3306 \
	          --env=MYSQL_ROOT_PASSWORD=<KODI_ROOT_PW> --env=MYSQL_USER=kodi \
	          --env=MYSQL_PASSWORD=kodi \
	          abrahammouse/armhf-alpine-mysql
	ExecStop=/storage/.kodi/addons/service.system.docker/bin/docker stop mysql

	[Install]
	WantedBy=multi-user.target

```
*Again, be sure to set the Kodi Root PW*

LibreELEC makes use of ```system.d``` for startup. We don't need to know the details here, but each file in the ```/storage/.config/system.d``` folder is parsed and run at startup. The file above tells ```system.d``` that we need to run this particular script *after* the ```service.system.docker.service``` has started, but before the ```kodi.service``` begins. This ensures that if kodi makes use of the MySQL service, we know it is started before Kodi trys to do so. The remainder of the file is fairly self explanatory, and simply calls the same command we looked at above.

After creating this file and rebooting, you should now have a Kodi box, with a MySQL Server process running in the background. The  principle can be followed above to install any available docker image (or any new  docker image you have created) as a background service, and thus turn your media box into a server!













