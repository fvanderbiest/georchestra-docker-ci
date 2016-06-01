# docker (deprecated)

This is outdated & unmaintained material, please refer to [https://github.com/georchestra/georchestra#install](https://github.com/georchestra/georchestra#install)

This repository contains the [docker](https://www.docker.com/) and [compose](https://docs.docker.com/compose/) configuration files required to build a basic [geOrchestra](http://www.georchestra.org) SDI instance.

There are two branches:
 * ```master``` is for a regular geOrchestra setup with GeoServer security,
 * ```geofence``` installs the GeoFence webapp and a modified version of GeoServer which includes the GeoFence probe.  

The goal is to setup a dev or demo server, but this is **NOT** for production use.  
If you need a [provisioning](https://docs.vagrantup.com/v2/provisioning/ansible.html) tool, have a look at this contributed [geOrchestra ansible playbook](https://github.com/landryb/georchestra-ansible).

The webapps are pulled from our [continuous integration](https://sdi.georchestra.org/ci/), which builds the complete stack every night from the master branch. You've been warned: it might not work at all !

## pre-requisites

Install a recent docker-engine: read https://blog.docker.com/2015/07/new-apt-and-yum-repos/ and:
```
sudo apt-get remove --purge docker.io
sudo apt-get remove --purge lxc-docker
sudo apt-get install docker-engine
```

Install Compose:
```
sudo -i
curl -L https://github.com/docker/compose/releases/download/1.5.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
exit
```

Create the directories on the host for the tmp and data volumes:
```
mkdir -p ~/docker/tmp ~/docker/data ~/docker/datadir && chmod -R a+rw ~/docker/*
```
The ```~/docker/data``` folder is mounted on ```/data``` in the GeoServer container. Copy some of your geodata inside !

Populate the geOrchestra config dir:
```
cd ~/docker && git clone -b docker https://github.com/georchestra/config.git datadir
```

Clone the repository:
```
git clone --recursive https://github.com/georchestra/docker.git ~/docker_georchestra
```

## build & run

Run the stack with:
```
cd ~/docker_georchestra && docker-compose up
```

Register the IP of the front server in your host machine with this one-liner:
```
sudo sed -i '/georchestra.mydomain.org/d' /etc/hosts && \
sudo sh -c "echo `docker inspect --format {{.NetworkSettings.IPAddress}} docker_apache_1` georchestra.mydomain.org >> /etc/hosts"
```

Open [http://georchestra.mydomain.org/header/](http://georchestra.mydomain.org/header/) in your browser and voila !  
Login with ```testadmin``` / ```testadmin```.

Once you're done with it, you can stop the containers with CTRL + C

## customizing the images

In case you make changes to the Dockerfiles, and before running again ```docker-compose up```, remember you have to:
 - stop the containers with CTRL + C
 - remove the containers with ```docker-compose rm```
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
