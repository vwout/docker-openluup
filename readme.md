# docker-openluup

[![](https://images.microbadger.com/badges/image/vwout/openluup.svg)](https://microbadger.com/images/vwout/openluup "Docker image layers provided by microbadger.com")
[![](https://images.microbadger.com/badges/version/vwout/openluup.svg)](https://microbadger.com/images/vwout/openluup "Docker image version provided by microbadger.com")
[![](https://images.microbadger.com/badges/commit/vwout/openluup.svg)](https://microbadger.com/images/vwout/openluup "Docker image git commit provided by microbadger.com")

A [Dockerfile](blob/master/openluup/Dockerfile) and [Composefile](blob/master/docker-compose.yml) for creation of a Debian based openluup container.

[OpenLuup](https://github.com/akbooer/openLuup) is a pure Lua open-source emulation of the [Vera](http://getvera.com/) Luup environment.

The Dockerized openluup environment is based on the openLuup installation script and starts with a running OpenLuup instance on port 3480 using a modified version of the openluup_reload script. This modified script adds a hook that captures the stop signal send to the container and calls the luup stop API call to gracefully shutdown a container.

OpenLuup is installed in ```/etc/cmh-ludl/```. The folders ```/etc/cmh-ludl/backup/``` and ```/etc/cmh-lu``` are shared as volumes to store user files or retrieve backups of the user_data JSON file. To provide a custom user_data.json file, mount it as ```/etc/cmh-ludl/user_data.json```, or set the path to an alternative user_data JSON file using the environment variable ```USER_DATA_JSON```.
