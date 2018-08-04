# docker-openluup
A Dockerfile and Composefile for creation of a Debian based openluup container.

[OpenLuup](https://github.com/akbooer/openLuup) is a pure Lua open-source emulation of the [Vera](http://getvera.com/) Luup environment.

The Dockerized openluup environment is based on the upenLuup installation script and starts with a running OpenLuup instance on port 3480 using a modified version of the openluup_reload script. This modified script adds a hook that captures the stop signal send to the container and calls the luup stop API call to gracefully shutdown a container.

OpenLuup is installed in ```/etc/cmh-ludl/```. The folders ```/etc/cmh-ludl/backup/``` and ```/etc/cmh-lu``` are shared as volumes to store user files or retrieve backups of the user_data JSON file. To provide a custom user_data.json file, mount it as ```/etc/cmh-ludl/user_data.json```.
