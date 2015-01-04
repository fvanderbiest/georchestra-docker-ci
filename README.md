# docker + fig

This repository contains the ```Dockerfile```s and ```fig.yml``` required to orchestrate docker containers in order to setup a basic [geOrchestra](http://www.georchestra.org) SDI instance.

## pre-requisites

Install a recent Docker version:

```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9
sudo sh -c "echo deb https://get.docker.com/ubuntu docker main /etc/apt/sources.list.d/docker.list"
sudo apt-get update
sudo apt-get install lxc-docker
```

Install Fig:
```
curl -L https://github.com/docker/fig/releases/download/1.0.1/fig-`uname -s`-`uname -m` > /usr/local/bin/fig; chmod +x /usr/local/bin/fig
```

Create directories on the host for logs (volumes):
```
mkdir -p ~/docker/volumes/georchestra/postgresql_logs \
    ~/docker/volumes/georchestra/apache_logs \
    ~/docker/volumes/georchestra/proxycas_logs \
    ~/docker/volumes/georchestra/mapfishapp_logs \
    ~/docker/volumes/georchestra/ldapadmin_logs \
    ~/docker/volumes/georchestra/geoserver_logs && \
    chmod -R a+rw ~/docker/volumes/georchestra/*
```

## build & run

Run the stack with:
```
fig up
```
Stop it with CTRL + C

To get the IP of the front server:
```
docker inspect docker_apache_1 | grep IPAddress
```
It should be something like 172.17.0.xxx

In your host machine:
```
echo "172.17.0.xxx       vm-georchestra" >> /etc/hosts
```

Open http://vm-georchestra/header/ in your browser and voila !


## customizing the images

In case you make changes to the Dockerfiles, and before running again ```fig up```, remember you have to:
 - remove the containers with ```fig rm```
 - remove the previous images with eg ```docker rmi -f docker_proxycas``` (in case you modified proxycas/Dockerfile)


