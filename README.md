# docker + fig

This repository contains the [docker](https://www.docker.com/) and [fig](http://www.fig.sh/) configuration files required to build a basic [geOrchestra](http://www.georchestra.org) SDI instance.

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
curl -L https://github.com/docker/fig/releases/download/1.0.1/fig-`uname -s`-`uname -m` > /usr/local/bin/fig; chmod +x /usr/local/bin/fig
```
or
```
sudo pip install -U fig
```

Create the directories on the host for the logs (volumes):
```
mkdir -p ~/docker/volumes/georchestra/postgresql_logs \
    ~/docker/volumes/georchestra/apache_logs \
    ~/docker/volumes/georchestra/catalogapp_logs \
    ~/docker/volumes/georchestra/extractorapp_logs \
    ~/docker/volumes/georchestra/downloadform_logs \
    ~/docker/volumes/georchestra/proxycas_logs \
    ~/docker/volumes/georchestra/mapfishapp_logs \
    ~/docker/volumes/georchestra/ldapadmin_logs \
    ~/docker/volumes/georchestra/analytics_logs \
    ~/docker/volumes/georchestra/geonetwork_logs \
    ~/docker/volumes/georchestra/geowebcache_logs \
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
docker inspect --format {{.NetworkSettings.IPAddress}} docker_apache_1
```
It should be something like 172.17.0.xx

Register it in your host machine:
```
echo "172.17.0.xx       vm-georchestra" >> /etc/hosts
```

... or with a one-liner:
```
sudo sed -i '/vm-georchestra/d' /etc/hosts && \
sudo sh -c "echo `docker inspect --format {{.NetworkSettings.IPAddress}} docker_apache_1` vm-georchestra >> /etc/hosts"
```

Open [http://vm-georchestra/header/](http://vm-georchestra/header/) in your browser and voila !


## customizing the images

In case you make changes to the Dockerfiles, and before running again ```fig up```, remember you have to:
 - stop the containers with CTRL + C
 - remove the containers with ```fig rm```
 - remove the previous images with eg ```docker rmi -f docker_proxycas``` (in case you modified proxycas/Dockerfile)

## debugging

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

