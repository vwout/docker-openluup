# docker-openluup

A Dockerfile and [Composefile](blob/master/docker-compose.yml) for creation of a [Debian](blob/master/openluup/Dockerfile) or [Alpine](blob/master/openluup-alpine/Dockerfile) based openluup container.

[OpenLuup](https://github.com/akbooer/openLuup) is a pure Lua open-source emulation of the [Vera](http://getvera.com/) Luup environment.

The Dockerized openluup environment is based on the openLuup installation script and starts with a running OpenLuup instance on port 3480 using a modified version of the openluup_reload script. This modified script adds a hook that captures the stop signal send to the container and calls the luup stop API call to gracefully shutdown a container.

The docker image of openLuup is available from Docker Hub as [vwout/openluup](https://hub.docker.com/r/vwout/openluup/) and comes in two flavors: based on Debian (tagged 'slim') and based on Alpine (tagged 'alpine'). This image contains the master branch code of openLuup.

| Base image | Label | Layers | Vcs Revision |
|------------|-------|--------|--------------|
| Debian stretch (slim) | [![](https://images.microbadger.com/badges/version/vwout/openluup:slim.svg)](https://microbadger.com/images/vwout/openluup:slim "Docker image version for Debian") | [![](https://images.microbadger.com/badges/image/vwout/openluup:slim.svg)](https://microbadger.com/images/vwout/openluup:slim "Docker image layers for Debian") | [![](https://images.microbadger.com/badges/commit/vwout/openluup:slim.svg)](https://microbadger.com/images/vwout/openluup:slim "Docker image git commit for Debian") |
| Alpine 3.8 | [![](https://images.microbadger.com/badges/version/vwout/openluup:alpine.svg)](https://microbadger.com/images/vwout/openluup:alpine "Docker image version for alpine") | [![](https://images.microbadger.com/badges/image/vwout/openluup:alpine.svg)](https://microbadger.com/images/vwout/openluup:alpine "Docker image layers for alpine") | [![](https://images.microbadger.com/badges/commit/vwout/openluup:alpine.svg)](https://microbadger.com/images/vwout/openluup:alpine "Docker image git commit for alpine") |

## Running
To start openLuup, invoke docker and add options like volume mounts (see below) to your liking:

    docker run -p 3480:3480 vwout/openluup

For ease of use, the docker-compose.yml file defines the openLuup image in two variants (using the master and development branch of openLuup) and contains a definition for mapping ports, binding a local directory for plugin development and mounting named volumes.

    docker-compose up openluup-development

## Configuration
The image runs openLuup with AltUI out of the box. Confguration is easiest via AltUI, which is available on http://localhost:3480/, which redirects to http://localhost:3480/data_request?id=lr_ALTUI_Handler&command=home#.
Additionally, openLuup provides a console interface for easy operational access at http://localhost:3480/console.

OpenLuup is installed in ```/etc/cmh-ludl/```. This folder and the folders ```/etc/cmh-ludl/backup/``` and ```/etc/cmh-lu``` are shared as volumes to store user files or retrieve backups of the user_data JSON file.

### Environment
- ```USER_DATA_JSON```: To provide a custom user_data.json file, mount it as ```/etc/cmh-ludl/user_data.json```, or set the path to an alternative user_data JSON file using the environment variable ```USER_DATA_JSON```. Setting the environment variable will load the provided userdata. openLuup will not update the userdata to the file. The json is only used during startup.
To obtain the configuration, use ```user_data.json``` from ```/etc/cmh-ludl/```, or pick a backup from ```/etc/cmh-ludl/backup/```.
- ```TZ```: The timezone used in the container can be set using the ```TZ``` environment variable. The image defaults to ```UTC```. Both timezone names (e.g. 'Europe/Amsterdam') and UTC offsets (e.g. 'GMT+2').

## Persisting configuration
To keep the openLuup configuration and plugin data even when removing the openLuup image, use volumes. The image defines the volumes as mentioned above.

For easiest operation, use the provided docker-compose file. It creates 3 named volumes (openluup-env, openluup-logs, openluup-backups) and contain the openLuup environment, logs and backups respectively.

You can also do this manually.
Start by creating docker volumes:

    docker volume create openluup-env
    docker volume create openluup-logs
    docker volume create openluup-backups

Create an openLuup container, e.g. based on Alpine linux and mount the created (still empty) volumes:

    docker run -d \
           -v openluup-env:/etc/cmh-ludl/ \
           -v openluup-logs:/etc/cmh-ludl/logs/ \
           -v openluup-backups:/etc/cmh-ludl/backup/ \
           -p 3480:3480
           vwout/openluup:alpine

Any configuration change that you apply on openLuup or AltUI, like installing and configuring plugins, will be saved on the ```openluup-env``` volume. This volume will not be removed when the container stops or the image is removed. The data is only deleted upon a ```docker volume rm``` action.

## Credits
Credits for the alpine based lua (and luarocks) image go to Andriy Kornatskyy for his [docker-library for lua](https://github.com/akornatskyy/docker-library). The alpine image for openLuup is using alpine directly instead of using [akorn/luarocks](https://hub.docker.com/r/akorn/luarocks/) because that lua-alpine base image is compiled for posix instead of linux, which prevents the usage of modules installed via luarocks, and also for size optimization reasons.
