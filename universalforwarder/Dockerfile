FROM ubuntu:latest

MAINTAINER Denis Gladkikh <docker-splunk@denis.gladkikh.email>

ENV SPLUNK_PRODUCT universalforwarder
ENV SPLUNK_VERSION 6.3.2
ENV SPLUNK_BUILD aaff59bb082c
ENV SPLUNK_FILENAME splunkforwarder-${SPLUNK_VERSION}-${SPLUNK_BUILD}-Linux-x86_64.tgz

ENV SPLUNK_HOME /opt/splunk
ENV SPLUNK_GROUP splunk
ENV SPLUNK_USER splunk
ENV SPLUNK_BACKUP_DEFAULT_ETC /var/opt/splunk
ENV SPLUNK_FORWARD_SERVER localhost:9997

# add splunk:splunk user
RUN groupadd -r ${SPLUNK_GROUP} \
    && useradd -r -m -g ${SPLUNK_GROUP} ${SPLUNK_USER}

# make the "en_US.UTF-8" locale so splunk will be utf-8 enabled by default
RUN apt-get update && apt-get install -y locales \
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG en_US.utf8

# Download official Splunk release, verify checksum and unzip in /opt/splunk
# Also backup etc folder, so it will be later copied to the linked volume
RUN apt-get install -y wget \
    && mkdir -p ${SPLUNK_HOME} \
    && wget -qO /tmp/${SPLUNK_FILENAME} https://download.splunk.com/products/splunk/releases/${SPLUNK_VERSION}/${SPLUNK_PRODUCT}/linux/${SPLUNK_FILENAME} \
    && wget -qO /tmp/${SPLUNK_FILENAME}.md5 https://download.splunk.com/products/splunk/releases/${SPLUNK_VERSION}/${SPLUNK_PRODUCT}/linux/${SPLUNK_FILENAME}.md5 \
    && (cd /tmp && md5sum -c ${SPLUNK_FILENAME}.md5) \
    && tar xzf /tmp/${SPLUNK_FILENAME} --strip 1 -C ${SPLUNK_HOME} \
    && rm /tmp/${SPLUNK_FILENAME} \
    && rm /tmp/${SPLUNK_FILENAME}.md5 \
    && apt-get purge -y --auto-remove wget \
    && mkdir -p /var/opt/splunk \
    && cp -R ${SPLUNK_HOME}/etc ${SPLUNK_BACKUP_DEFAULT_ETC} \
    && rm -fR ${SPLUNK_HOME}/etc \
    && chown -R ${SPLUNK_USER}:${SPLUNK_GROUP} ${SPLUNK_HOME} \
    && chown -R ${SPLUNK_USER}:${SPLUNK_GROUP} ${SPLUNK_BACKUP_DEFAULT_ETC} \
    && apt-get install -y default-jre \
    && rm -rf /var/lib/apt/lists/*

EXPOSE 9997 8080

COPY lili_log.log /var/log/lili_log.log
COPY flower-0.0.1-SNAPSHOT.jar /sbin/flower.jar

#RUN chown ${SPLUNK_USER}:${SPLUNK_GROUP} /sbin/flower.jar
RUN chmod u=rw,g=rw,o=r /sbin/flower.jar

COPY entrypoint.sh /sbin/entrypoint.sh
RUN chmod +x /sbin/entrypoint.sh

# Ports Splunk Daemon, Network Input, HTTP Event Collector
EXPOSE 8089/tcp 1514 8088/tcp

WORKDIR /opt/splunk

# Configurations folder, var folder for everyting (indexes, logs, kvstore)
VOLUME [ "/opt/splunk/etc", "/opt/splunk/var"]

ENTRYPOINT ["/sbin/entrypoint.sh"]
CMD ["start-service"]
