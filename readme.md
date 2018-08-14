# docker-openluup

A [Dockerfile](blob/master/openluup/Dockerfile) and [Composefile](blob/master/docker-compose.yml) for creation of a Debian based openluup container.

[OpenLuup](https://github.com/akbooer/openLuup) is a pure Lua open-source emulation of the [Vera](http://getvera.com/) Luup environment.

The Dockerized openluup environment is based on the openLuup installation script and starts with a running OpenLuup instance on port 3480 using a modified version of the openluup_reload script. This modified script adds a hook that captures the stop signal send to the container and calls the luup stop API call to gracefully shutdown a container.

The docker image of openLuup is available from Docker Hub as [vwout/openluup](https://hub.docker.com/r/vwout/openluup/). This image contains the master branch code of openLuup.

| Layers | Label | Vcs Revision |
|--------|-------|--------------|
| [![](https://images.microbadger.com/badges/image/vwout/openluup.svg)](https://microbadger.com/images/vwout/openluup "Docker image layers provided by microbadger.com") | [![](https://images.microbadger.com/badges/version/vwout/openluup.svg)](https://microbadger.com/images/vwout/openluup "Docker image version provided by microbadger.com") | [![](https://images.microbadger.com/badges/commit/vwout/openluup.svg)](https://microbadger.com/images/vwout/openluup "Docker image git commit provided by microbadger.com") |

## Running
To start openLuup, invoke docker and add options like volume mounts (see below) to your liking:

    docker run -p 3480:3480 vwout/openluup

For ease of use, the docker-compose.yml file defines the openLuup image in two variants (using the master and development branch of openLuup) and contains a definition for mapping ports, binding a local directory for plugin development and mounting named volumes.

    docker-compose up openluup-development

## Configuration
The image runs openLuup with AltUI out of the box. Confguration is easiest via AltUI, which is available on http://localhost:3480/, which redirects to http://localhost:3480/data_request?id=lr_ALTUI_Handler&command=home#.
Additionally, openLuup provides a console interface for easy operational access at http://localhost:3480/console.

OpenLuup is installed in ```/etc/cmh-ludl/```. This folder and the folders ```/etc/cmh-ludl/backup/``` and ```/etc/cmh-lu``` are shared as volumes to store user files or retrieve backups of the user_data JSON file.
To provide a custom user_data.json file, mount it as ```/etc/cmh-ludl/user_data.json```, or set the path to an alternative user_data JSON file using the environment variable ```USER_DATA_JSON```. Setting the environment variable will load the provided userdata. openLuup will not update the userdata to the file. The json is only used during startup.
To obtain the configuration, use ```user_data.json``` from ```/etc/cmh-ludl/```, or pick a backup from ```/etc/cmh-ludl/backup/```.

## Persisting configuration
To keep the openLuup configuration and plugin data even when removing the openLuup image, use volumes. The image defines the volumes as mentioned above.

For easiest operation, use the provided docker-compose file. It creates 3 named volumes (openluup-env, openluup-logs, openluup-backups) and contain the openLuup environment, logs and backups respectively.
