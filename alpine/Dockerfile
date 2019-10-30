FROM alpine:3.8

LABEL maintainer="Daniel Sullivan <https://github.com/mumblepins>"

ARG SYSLOG_VERSION="3.20.1"
ARG BUILD_CORES=2
RUN echo 'http://mirrors.ustc.edu.cn/alpine/v3.8/main' > /etc/apk/repositories \
    && echo 'http://mirrors.ustc.edu.cn/alpine/v3.8/community' >>/etc/apk/repositories \
    && apk add --no-cache \
    glib \
    pcre \
    eventlog \
    openssl \
    tini \
    && apk add --no-cache --virtual .build-deps \
    curl \
    alpine-sdk \
    glib-dev \
    pcre-dev \
    eventlog-dev \
    openssl-dev \
    && set -ex \
    && cd /tmp \
    && curl -sSL "https://github.com/balabit/syslog-ng/releases/download/syslog-ng-${SYSLOG_VERSION}/syslog-ng-${SYSLOG_VERSION}.tar.gz" \
        | tar xz \
    && cd "syslog-ng-${SYSLOG_VERSION}" \
    && ./configure -q --prefix=/usr \
    && make -j $BUILD_CORES \
    && make install \
    && rm -rf /tmp/* \
    && apk del --no-cache .build-deps \
    && mkdir -p /etc/syslog-ng /var/run/syslog-ng /var/log/syslog-ng

#COPY ./etc /etc/
COPY syslog-ng.conf startup.sh /app/

RUN chmod +x /app/startup.sh \
    # Replace the system() source because inside Docker we can't access /proc/kmsg.
    # https://groups.google.com/forum/#!topic/docker-user/446yoB0Vx6w
    && sed -i -E 's/^(\s*)system\(\);/\1unix-stream("\/dev\/log");/' /etc/syslog-ng/syslog-ng.conf \
    # Uncomment 'SYSLOGNG_OPTS="--no-caps"' to avoid the following warning:
    # syslog-ng: Error setting capabilities, capability management disabled; error='Operation not permitted'
    # http://serverfault.com/questions/524518/error-setting-capabilities-capability-management-disabled#
    && sed -i 's/^#\(SYSLOGNG_OPTS="--no-caps"\)/\1/g' /etc/default/syslog

# Build-time metadata as defined at http://label-schema.org
ARG BUILD_DATE
ARG VCS_REF
ARG VCS_URL
ARG VERSION

LABEL org.label-schema.build-date=$BUILD_DATE \
      org.label-schema.name="syslog-ng-alpine" \
      org.label-schema.description="Minimal Syslog Build on Alpine" \
      org.label-schema.url="https://hub.docker.com/r/mumblepins/syslog-ng-alpine/" \
      org.label-schema.vcs-ref=$VCS_REF \
      org.label-schema.vcs-url=$VCS_URL \
      org.label-schema.version=$VERSION \
      org.label-schema.schema-version="1.0" \
      org.label-schema.docker.cmd="docker run -d -p 514:514/udp -v /var/log/syslog-ng:/var/log/syslog-ng --name syslog-ng mumblepins/syslog-ng-alpine"

VOLUME ["/var/log/syslog-ng", "/var/run/syslog-ng", "/etc/syslog-ng"]

EXPOSE 514/udp 601/tcp 6514/tcp

ENTRYPOINT ["tini", "--"]

CMD ["/bin/sh", "-c", "/app/startup.sh && exec /usr/sbin/syslog-ng -F -f /etc/syslog-ng/syslog-ng.conf"]
