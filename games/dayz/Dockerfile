FROM        --platform=$TARGETOS/$TARGETARCH debian:bookworm-slim

LABEL       author="David Wolfe (Red-Thirten)" maintainer="red_thirten@yahoo.com"

LABEL       org.opencontainers.image.source="https://github.com/perfheberg/yolks"
LABEL       org.opencontainers.image.licenses=MIT

## Update base packages and install dependencies
ENV         DEBIAN_FRONTEND=noninteractive
RUN         dpkg --add-architecture i386 \
            && apt update \
            && apt upgrade -y \
            && apt install -y \
                curl \
                tzdata \
                locales \
                iproute2 \
                gettext-base \
                ca-certificates \
                libssl-dev \
                lib32gcc-s1 \
                libsdl2-2.0-0 \
                libsdl2-2.0-0:i386 \
                libstdc++6 \
                libstdc++6:i386 \
                lib32stdc++6 \
                libcap2 \
                libnss-wrapper \
                tini

## install rcon client (bercon)
RUN         cd /tmp/ \
            && curl -sSL https://github.com/WoozyMasta/bercon/releases/download/1.0.0/bercon > bercon \
            && mv bercon /usr/local/bin/ \
            && chmod +x /usr/local/bin/bercon

## Configure locale
RUN         update-locale lang=en_US.UTF-8 \
            && dpkg-reconfigure --frontend noninteractive locales

## Prepare NSS Wrapper for the entrypoint as a workaround for DayZ requiring a valid UID
ENV         NSS_WRAPPER_PASSWD=/tmp/passwd NSS_WRAPPER_GROUP=/tmp/group
RUN         touch ${NSS_WRAPPER_PASSWD} ${NSS_WRAPPER_GROUP} \
            && chgrp 0 ${NSS_WRAPPER_PASSWD} ${NSS_WRAPPER_GROUP} \
            && chmod g+rw ${NSS_WRAPPER_PASSWD} ${NSS_WRAPPER_GROUP}
ADD         passwd.template /passwd.template

## Setup user and working directory
RUN         useradd -m -d /home/container -s /bin/bash container
USER        container
ENV         USER=container HOME=/home/container
WORKDIR     /home/container

STOPSIGNAL  SIGINT

## Copy over and execute entrypoint.sh via Tini
COPY        --chown=container:container ./entrypoint.sh /entrypoint.sh
RUN         chmod +x /entrypoint.sh
ENTRYPOINT  ["/usr/bin/tini", "-g", "--"]
CMD         ["/entrypoint.sh"]
