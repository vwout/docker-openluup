##############################################################################
# Use Alpine 3.11 image as base
FROM alpine:3.11

MAINTAINER "Vwout <vwout@users.noreply.github.com>"

ARG LUA_VERSION=5.1.5
ARG LUAROCKS_VERSION=3.0.0
ARG LUACOAP_RELEASE=aa58ad22fbe3ea9ff83e5f1b93af968e825bc956
ARG INSTALL_ROCKS="luasocket luafilesystem luasec luabit32 luabitop"

COPY lua-${LUA_VERSION}-so.patch /

#
# Add lua and luarocks to install lua modules (a.k.a. rocks)
# To add lua and luarocks, a gcc-based development environment
# is temporarily installed. These dependencies are removed
# afterwards to save space.
RUN set -ex \
    \
    && apk add --no-cache \
        ca-certificates \
        curl \
        openssl \
        readline \
        tzdata \
    \
    && apk add --no-cache --virtual .build-deps \
        gcc \
        git \
        cmake \
        libtool \
        autoconf-archive \
        automake \
        bsd-compat-headers \
        musl-dev \
        make \
        ncurses-dev \
        openssl-dev \
        readline-dev \
    \
    && wget -c https://www.lua.org/ftp/lua-${LUA_VERSION}.tar.gz \
        -O - | tar -xzf - \
    \
    && cd lua-${LUA_VERSION} \
    && patch -p1 < /lua-${LUA_VERSION}-so.patch \
    && rm /lua-${LUA_VERSION}-so.patch \
    && make -j"$(nproc)" linux \
    && make install \
    && cd / \
    && rm -rf /lua-${LUA_VERSION} \
    \
    && ln -s /usr/local/bin/lua /usr/local/bin/lua5.1 \
    \
    && git clone https://github.com/vwout/luacoap \
    && cd luacoap \
    && git checkout ${LUACOAP_RELEASE} \
    && mkdir build \
    && cd build \
    && cmake -DLUA_VERSION=5.1 .. \
    && make \
    && make install \
    && cd / \
    && rm -rf /luacoap \
    \
    && wget -c https://github.com/luarocks/luarocks/archive/v${LUAROCKS_VERSION}.tar.gz \
        -O - | tar -xzf - \
    \
    && cd luarocks-${LUAROCKS_VERSION} \
    && ./configure --with-lua=/usr/local \
    && make build \
    && make install \
    && cd .. \
    && rm -rf luarocks-${LUAROCKS_VERSION} \
    \
    && for rock in ${INSTALL_ROCKS}; do \
        luarocks install ${rock}; \
       done \
    \
    && apk del .build-deps


#
# Build arguments to install openluup and set metadata labels
ARG BUILD_DATE
ARG VCS_REF
ARG IMAGE_VERSION="0.1.0"
ARG OPENLUUP_FORK="akbooer"
ARG OPENLUUP_BRANCH="master"

# Change the environment variable USER_DATA to point to a different file for loading an alternative configuration
ENV USER_DATA_JSON user_data.json

# Set default timezone to UTC
ENV TZ UTC

# Install and configure openLuup
RUN mkdir -p /etc/cmh-ludl/ /etc/cmh-ludl/backup /etc/cmh-lu/ \
    \
    && cd /etc/cmh-ludl/ \
    \
    && wget https://github.com/$OPENLUUP_FORK/openLuup/raw/$OPENLUUP_BRANCH/Utilities/openLuup_install.lua \
    && lua openLuup_install.lua $OPENLUUP_BRANCH \
    && rm openLuup_install.lua latest.tar.gz \
    \
    && ln -s /etc/cmh-ludl/logs /var/log/cmh \
    && (curl --retry 5 --connect-timeout 1 --retry-delay 0 --max-time 1 "http://localhost:3480/data_request?id=action&DeviceNum=3&serviceId=urn:upnp-org:serviceId:altui1&action=Reset" || true) \
    \
    && wget https://github.com/pkulchenko/MobDebug/raw/master/src/mobdebug.lua

WORKDIR /etc/cmh-ludl/

# Add openLuup startup script and patch AltUI PluginConfig loading
ADD openLuup_reload_for_docker /etc/cmh-ludl/
RUN sed -i -e 's/\r$//' openLuup_reload_for_docker \
    && \
    chmod +x openLuup_reload_for_docker

# Expose the altUI web interface and console, running on port 3480 by default,
# the interal SMTP server added in openLuup (v18.3.14), running on port 2525 by default
# and the internal POP3 server, running on port 11011 by default
EXPOSE 3480 2525 11011

VOLUME ["/etc/cmh-ludl/", "/etc/cmh-ludl/backup/", "/etc/cmh-ludl/logs/", "/etc/cmh-lu/"]

ENTRYPOINT /bin/sh /etc/cmh-ludl/openLuup_reload_for_docker $USER_DATA_JSON

# Metadata
LABEL maintainer="vwout <vwout@users.noreply.github.com>" \
      org.label-schema.name="openLuup ($OPENLUUP_BRANCH)" \
      org.label-schema.description="openLuup on Alpine with AltUI" \
      org.label-schema.vcs-url="https://github.com/vwout/docker-openluup.git" \
      org.label-schema.vcs-ref=$VCS_REF \
      org.label-schema.build-date=$BUILD_DATE \
      org.label-schema.schema-version="$IMAGE_VERSION"
