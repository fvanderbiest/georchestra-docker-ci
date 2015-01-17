# docker + fig

This repository contains the [docker](https://www.docker.com/) and [fig](http://www.fig.sh/) configuration files required to build a basic [geOrchestra](http://www.georchestra.org) SDI instance with GeoFence.

The goal is to setup a dev or demo server, but this is **NOT** for production use.  
If you need a [provisioning](https://docs.vagrantup.com/v2/provisioning/ansible.html) tool, have a look at this contributed [geOrchestra ansible playbook](https://github.com/landryb/georchestra-ansible).

## pre-requisites

Install a recent Docker version:
```
sudo apt-get remove docker.io
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9
sudo sh -c "echo deb https://get.docker.com/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
sudo apt-get update
sudo apt-get install lxc-docker
```

Install Fig:
```
sudo apt-get install python-pip
sudo pip install -U fig
```

Create the directories on the host for the tmp and data volumes:
```
mkdir -p ~/docker/tmp ~/docker/data && chmod -R a+rw ~/docker/*
```

Clone the repository:
```
git clone -b geofence --recursive https://github.com/georchestra/docker.git ~/docker_georchestra
```

## build & run

Run the stack with:
```
cd ~/docker_georchestra && fig up
```

Register the IP of the front server in your host machine with this one-liner:
```
sudo sed -i '/vm-georchestra/d' /etc/hosts && \
sudo sh -c "echo `docker inspect --format {{.NetworkSettings.IPAddress}} docker_apache_1` vm-georchestra >> /etc/hosts"
```

Open [http://vm-georchestra/header/](http://vm-georchestra/header/) in your browser and voila !

Note that you can stop the containers with CTRL + C

## customizing the images

In case you make changes to the Dockerfiles, and before running again ```fig up```, remember you have to:
 - stop the containers with CTRL + C
 - remove the containers with ```fig rm```
 - remove the previous images with eg ```docker rmi -f docker_proxycas``` (in case you modified proxycas/Dockerfile)

## debugging

Inspect the logs:
```
tail -f ~/docker/tmp/security-proxy.log
```

In order to see what's going on in the containers, open a console with, eg:
```
docker exec -it docker_proxycas_1 bash
```
... where ```docker_proxycas_1``` is the name of the running container (retrieve it with ```docker ps```).

For the network:
```
sudo tcpdump -X -i docker0 tcp port XXXX
```
... where XXXX might be 8180, 8280, 8281, 8282, 8380, etc... depending on which webapp you have to debug.

