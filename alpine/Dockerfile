ARG IMAGE_ALPINE_NAME
ARG IMAGE_ALPINE_VERSION


FROM ${IMAGE_ALPINE_NAME}:${IMAGE_ALPINE_VERSION}

RUN apk add --no-cache ca-certificates libstdc++ su-exec libpq
RUN set -eux; \
    addgroup -g 9987 ts3server; \
    adduser -u 9987 -Hh /var/ts3server -G ts3server -s /sbin/nologin -D ts3server; \
    install -d -o ts3server -g ts3server -m 775 /var/ts3server /var/run/ts3server /opt/ts3server

ENV PATH "${PATH}:/opt/ts3server"

ARG TEAMSPEAK_CHECKSUM
ARG TEAMSPEAK_VERSION
ARG TEAMSPEAK_URL=https://files.teamspeak-services.com/releases/server/${TEAMSPEAK_VERSION}/teamspeak3-server_linux_alpine-${TEAMSPEAK_VERSION}.tar.bz2
ARG TIMEZONE=Europe/Warsaw

RUN set -eux; \
    apk add --no-cache --virtual .fetch-deps tar; \
    # install tzdata
    apk add --no-cache tzdata; \
    wget "${TEAMSPEAK_URL}" -O server.tar.bz2; \
    echo "${TEAMSPEAK_CHECKSUM} *server.tar.bz2" | sha256sum -c -; \
    mkdir -p /opt/ts3server; \
    tar -xf server.tar.bz2 --strip-components=1 -C /opt/ts3server; \
    rm server.tar.bz2; \
    apk del .fetch-deps; \
    mv /opt/ts3server/*.so /opt/ts3server/redist/* /usr/local/lib; \
    ldconfig /usr/local/lib

ENV TEAMSPEAK_VERSION=${TEAMSPEAK_VERSION}
ENV TZ=${TIMEZONE}

# setup directory where user data is stored
VOLUME /var/ts3server/
WORKDIR /var/ts3server/

#  9987 default voice
# 10011 server query
# 30033 file transport
EXPOSE 9987/udp 10011 30033

COPY alpine/entrypoint.sh /opt/ts3server

ENTRYPOINT [ "entrypoint.sh" ]
CMD [ "ts3server" ]
