# Use Debian 9 minimal image as base
FROM debian:stretch-slim
MAINTAINER vwout <vwout@users.noreply.github.com>

# Add required packages
RUN apt-get update && \
    apt-get install -y wget &&  \
    apt-get install -y lua5.1 lua-socket lua-filesystem lua-sec

# Build arguments to set metadata labels
ARG BUILD_DATE
ARG VCS_REF
ARG IMAGE_VERSION="0.0.1"
ARG OPENLUUP_BRANCH="master"

# Change the environment variable USER_DATA to point to a different file for loading an alternative configuration
ENV USER_DATA_JSON user_data.json

# Install and configure openLuup
RUN mkdir -p /etc/cmh-ludl/ /etc/cmh-ludl/backup /etc/cmh-lu/ && \
    cd /etc/cmh-ludl/ && \
    wget https://github.com/vwout/openLuup/raw/$OPENLUUP_BRANCH/Utilities/openLuup_install.lua && \
    lua5.1 openLuup_install.lua $OPENLUUP_BRANCH && \
    rm openLuup_install.lua latest.tar.gz && \
    (wget --tries 5 "http://localhost:3480/data_request?id=action&DeviceNum=3&serviceId=urn:upnp-org:serviceId:altui1&action=Reset" || true) && \
    wget https://github.com/pkulchenko/MobDebug/raw/master/src/mobdebug.lua

WORKDIR /etc/cmh-ludl/

# Add openLuup startup script and patch AltUI PluginConfig loading
ADD openLuup_reload_for_docker /etc/cmh-ludl/
RUN chmod +x openLuup_reload_for_docker

# Expose the altUI web interface
EXPOSE 3480

VOLUME ["/etc/cmh-ludl/backup/", "/etc/cmh-ludl/logs/", "/etc/cmh-lu/"]

ENTRYPOINT /etc/cmh-ludl/openLuup_reload_for_docker $USER_DATA_JSON

# Metadata
LABEL maintainer="vwout <vwout@users.noreply.github.com>" \
      org.label-schema.name="openLuup" \
      org.label-schema.description="openLuup on Debian with AltUI" \
      org.label-schema.vcs-url="https://github.com/vwout/docker-openluup.git" \
      org.label-schema.vcs-ref=$VCS_REF \
      org.label-schema.build-date=$BUILD_DATE \
      org.label-schema.schema-version="$IMAGE_VERSION"
