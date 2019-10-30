# Kenya GDC Internal GeoNetwork Installation

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
