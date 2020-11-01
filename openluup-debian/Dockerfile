# Use Debian 10 'Buster' minimal image as base for building luacoap
FROM debian:buster-slim as coap-build-build

ARG LUACOAP_RELEASE=443fa5f82921350b8cc5212834dd557af584ae8a

# Add required packages
RUN apt-get update && \
    apt-get install -y git cmake libtool autoconf-archive openssl libssl-dev lua5.1 lua5.1-dev


RUN git clone https://github.com/vwout/luacoap && \
    cd luacoap && \
    git checkout ${LUACOAP_RELEASE} && \
    mkdir build && \
    cd build && \
    cmake -DLUA_VERSION=5.1 .. && \
    make && \
    make install



##############################################################################
# Use Debian 10 'Buster' minimal image as base
FROM debian:buster-slim

MAINTAINER "Vwout <vwout@users.noreply.github.com>"

# Add required packages
RUN apt-get update && \
    apt-get install -y curl iputils-ping &&  \
    apt-get install -y openssl lua5.1 liblua5.1-0 lua-socket lua-filesystem lua-sec lua-bit32 lua-bitop lua-cjson lua-expat \
 && rm -rf /var/lib/apt/lists/*

# Build arguments to set metadata labels
ARG BUILD_DATE
ARG VCS_REF
ARG IMAGE_VERSION="0.1.0"
ARG OPENLUUP_FORK="akbooer"
ARG OPENLUUP_BRANCH="master"

# Change the environment variable USER_DATA to point to a different file for loading an alternative configuration
ENV USER_DATA_JSON user_data.json

# Install lua coap library from coap-build-build image
COPY --from=coap-build-build /usr/local/lib/lua/5.1/coap.so /usr/local/lib/lua/5.1/

# Install supplementary libraries
RUN mkdir -p /usr/local/lib/lua/5.1/ /usr/local/lib/lua/5.1/paho && \
    curl -L -sS "https://github.com/pkulchenko/MobDebug/raw/master/src/mobdebug.lua" -o /usr/local/lib/lua/5.1/mobdebug.lua && \
    curl -L -sS "http://regex.info/code/JSON.lua" -o /usr/local/lib/lua/5.1/JSON.lua && \
    curl -L -sS "http://dkolf.de/src/dkjson-lua.fsl/raw/dkjson.lua?name=16cbc26080996d9da827df42cb0844a25518eeb3" -o /usr/local/lib/lua/5.1/dkjson.lua && \
    curl -L -sS "https://raw.githubusercontent.com/iotrac/mqtt-lua/master/paho/mqtt.lua" -o /usr/local/lib/lua/5.1/mqtt_library.lua && \
    curl -L -sS "https://raw.githubusercontent.com/iotrac/mqtt-lua/master/paho/utility.lua" -o /usr/local/lib/lua/5.1/utility.lua

# Install and configure openLuup
RUN mkdir -p /etc/cmh-ludl/ /etc/cmh-ludl/backup /etc/cmh-lu/ && \
    cd /etc/cmh-ludl/ && \
    curl -L -sS https://github.com/$OPENLUUP_FORK/openLuup/raw/$OPENLUUP_BRANCH/Utilities/openLuup_install.lua --output openLuup_install.lua && \
    lua5.1 openLuup_install.lua $OPENLUUP_BRANCH && \
    rm openLuup_install.lua latest.tar.gz && \
    ln -s /etc/cmh-ludl/logs /var/log/cmh && \
    (curl --retry 5 "http://localhost:3480/data_request?id=action&DeviceNum=3&serviceId=urn:upnp-org:serviceId:altui1&action=Reset" --silent || true)

WORKDIR /etc/cmh-ludl/

# Add openLuup startup script and patch AltUI PluginConfig loading
ADD openLuup_reload_for_docker /etc/cmh-ludl/
RUN sed -i -e 's/\r$//' openLuup_reload_for_docker && \
    chmod +x openLuup_reload_for_docker

# Expose the altUI web interface and console, running on port 3480 by default,
# the interal SMTP server added in openLuup (v18.3.14), running on port 2525 by default
# and the internal POP3 server, running on port 11011 by default
EXPOSE 3480 2525 11011

VOLUME ["/etc/cmh-ludl/", "/etc/cmh-ludl/backup/", "/etc/cmh-ludl/logs/", "/etc/cmh-lu/"]

ENTRYPOINT /etc/cmh-ludl/openLuup_reload_for_docker $USER_DATA_JSON

# Metadata
LABEL maintainer="vwout <vwout@users.noreply.github.com>" \
      org.label-schema.name="openLuup ($OPENLUUP_BRANCH)" \
      org.label-schema.description="openLuup on Debian with AltUI" \
      org.label-schema.vcs-url="https://github.com/vwout/docker-openluup.git" \
      org.label-schema.vcs-ref=$VCS_REF \
      org.label-schema.build-date=$BUILD_DATE \
      org.label-schema.schema-version="$IMAGE_VERSION"
