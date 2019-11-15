# Kenya GDC Internal GeoNetwork Installation

![Kenya_GeoNetwork](Screen_Capture_191115.PNG "Kenya GeoNetwork")

## Install docker 

https://docs.docker.com/install/linux/docker-ce/ubuntu/ 

Open Terminal

Before you install Docker Engine - Community for the first time on a new host machine, you need to set up the Docker repository. Afterward, you can install and update Docker from the repository.

### Set up the repository - https://docs.docker.com/install/linux/docker-ce/ubuntu/#set-up-the-repository  

1.	Update the apt package index:
```bash 
$ sudo apt-get update
```
2.	Install packages to allow apt to use a repository over HTTPS:
```bash
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```
3.	Add Docker’s official GPG key:
```bash 
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
4.	Verify that you now have the key with the fingerprint 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88, by searching for the last 8 characters of the fingerprint.
```bash
$ sudo apt-key fingerprint 0EBFCD88
    
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```
5.	Use the following command to set up the stable repository.
```bash
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

### Install Docker Engine CE

1.	Update the apt package index.
```bash 
$ sudo apt-get update
```
2.	Install the latest version of Docker Engine - Community and containerd, or go to the next step to install a specific version:
```bash
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```
3.	Verify that Docker Engine - Community is installed correctly by running the hello-world image.
```bash
$ sudo docker run hello-world
```
## Install GeoNetwork Container - https://hub.docker.com/_/geonetwork 

1.	Pull docker container
```bash
$ sudo docker pull geonetwork
```
2.	Then we need to start GeoNetwork container, make auto restart, name it, publish the port (running docker on Linux, you may access geonetwork at http://localhost:8080/geonetwork. Otherwise, replace localhost by the address of your docker machine.), set the data directory, persist data. 
```bash
$ docker run --restart unless-stopped --name kenya-geonetwork -d -p 8080:8080 -e DATA_DIR=/var/lib/geonetwork_data -v /host/geonetwork-docker:/var/lib/geonetwork_data geonetwork
```

Now if go to http://localhost:8080/geonetwork GeoNetwork should load up. Depending on hardware might take 2 – 3 mins to spin up.

As a test 
1. Stop the container 
```bash 
$ sudo  docker stop kenya-geonetwork
```
2.	http://localhost:8080/geonetwork should stop working (click around for a bit)
```bash
$ sudo  docker start kenya-geonetwork
```
3.	Wait couple mins then http://localhost:8080/geonetwork should be working again. 

## Finally Enable Start on Reboot

2.	Make sure Docker Daemon starts on boot - https://docs.docker.com/install/linux/linux-postinstall//#configure-docker-to-start-on-boot
```bash
$ sudo systemctl enable docker
```

That should be it. Turn if Off/On to test.....

## Legacy Data Entry into GeoNetwork Walkthrough


### Introduction

The document gives a brief guide to initialisation of GeoNetwork with Asset Register data
It assumes an up to date functioning copy of GeoNetwork is running. 

### Workflow

-Convert Asset Register to Input Template
-Map input template to ISO19139/ISO19115 tags and generate XML files for each record
-Copy records to server (in mounted volume folder if using Docker)
```
/var/lib/geonetwork_data/…. 
```

Log into GeoNetwork
![F1](F1.PNG "F1") 
Go to Contribute > import new records 

![F2](F2.PNG "F2")  

![F3](F3.PNG "F3") 

```
/var/lib/geonetwork_data/data/kenxml_20191115
```

If don’t have suitable Group name available go to the Admin console and create one

![F4](F4.PNG "F4") 

### Thumbnails

Thumbnails had to be generated and location of files added to Asset Register Input Template

Installed https://www.bricelam.net/ImageResizer/
Used as per instructions on all raster files on W: drive
Used custom image size of maximum 600 pixels in either orientation, “fit” to maintain aspect ratio
![F5](F5.PNG "F5")  
Changed filename to append ”_Thumb” to original
Files stay in original folder locations & copied to open/less secure folder "metadata" so more users with less permissions can view the thumbnails. 
![F6](F6.PNG "F6") 
URL of file location added as new attribute to Asset register Template worksheet, can’t use W:\ or gdcs1\ 
MUST BE

http://gdcs1.gdc.com/spatialdata/raster/geological/sketch_plans/Report%20No.7_Thumb.jpg

## Backing Up

Using Docker we can commit docker container to an image. As we have a mounted volume we also need a copy of the data in that volume. 

### Saving the Container

```bash
gdc_admin@GDCS3:~/Documents$ sudo docker image ls
```

```bash
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
geonetwork          latest              c8d04e0920a2        3 weeks ago         812MB
hello-world         latest              fce289e99eb9        10 months ago       1.84kB
```

```bash
gdc_admin@GDCS3:~/Documents$ sudo docker container ls
```

```bash
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
0ba76a0dd671        geonetwork          "/entrypoint.sh cata…"   2 weeks ago         Up 6 days           0.0.0.0:8080->8080/tcp   kenya-geonetwork
```

```bash
gdc_admin@GDCS3:~/Documents$ sudo docker commit 0ba76a0dd671 kengn
```

```bash
sha256:1addea4861b47963f99a38bf27e468504c3148bf36e0c92595f3e7c51e9560a5
```

```bash
gdc_admin@GDCS3:~/Documents$ sudo docker image ls
```

```bash
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
kengn               latest              1addea4861b4        10 seconds ago      1.04GB
geonetwork          latest              c8d04e0920a2        3 weeks ago         812MB
hello-world         latest              fce289e99eb9        10 months ago       1.84kB
```

```bash
gdc_admin@GDCS3:~/Documents$ sudo docker save -o /home/gdc_admin/Documents/kenyagn.tar kengn
```

```bash
gdc_admin@GDCS3:~/Documents$ sudo chown gdc_admin:gdc_admin kenyagn.tar
```

### Saving the files 

Navigate to /host/ right click on the folder "geonetwork-docker" and compress to tar.xz

Copy kenyagn.tar and geonetwork-docker.tar.xz whereever required. 

![Kenya_GeoNetwork](Screen_Capture_191115_1.PNG "Kenya GeoNetwork")
![Kenya_GeoNetwork](Screen_Capture_191115_2.PNG "Kenya GeoNetwork")
![Kenya_GeoNetwork](Screen_Capture_191115_3.PNG "Kenya GeoNetwork")
