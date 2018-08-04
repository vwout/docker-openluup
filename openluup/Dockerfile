# Use Debian 9 minimal image as base
FROM debian:stretch-slim

RUN apt-get update && \
    apt-get install -y lua5.1 lua-socket lua-filesystem lua-sec && \
    apt-get install -y wget patch && \
    mkdir -p /etc/cmh-ludl/ /etc/cmh-ludl/backup /etc/cmh-lu/ && \
    cd /etc/cmh-ludl/ && \
    wget https://github.com/akbooer/openLuup/raw/master/Utilities/openLuup_install.lua && \
    lua5.1 openLuup_install.lua && \
    rm openLuup_install.lua latest.tar.gz

WORKDIR /etc/cmh-ludl/

# Add openLuup startup script and patch AltUI PluginConfig loading
ADD openLuup_reload_for_docker L_ALTUI.lua.patch /etc/cmh-ludl/
RUN chmod +x openLuup_reload_for_docker && \
    patch -p0 --binary < L_ALTUI.lua.patch && \
	rm L_ALTUI.lua.patch

# Expose the altUI web interface
EXPOSE 3480

VOLUME ["/etc/cmh-ludl/backup/", "/etc/cmh-lu"]

ENTRYPOINT ["/etc/cmh-ludl/openLuup_reload_for_docker", "user_data.json"]

# Set some metadata
LABEL maintainer="vwout"
