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

Create directories on the host for logs (volumes)

```
mkdir -p ~/docker/volumes/georchestra/postgresql_logs
mkdir -p ~/docker/volumes/georchestra/apache_logs
mkdir -p ~/docker/volumes/georchestra/proxycas_logs
mkdir -p ~/docker/volumes/georchestra/mapfishapp_logs
mkdir -p ~/docker/volumes/georchestra/geoserver_logs
chmod -R a+r ~/docker/volumes/georchestra/*
```

## build images

Run the stack with:
```
fig up
```

