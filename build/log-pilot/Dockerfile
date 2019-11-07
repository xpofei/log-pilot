FROM cargo.caicloud.xyz/library/debian:stretch-curl

COPY bin/log-pilot /opt/log-pilot/bin/log-pilot
COPY assets/filebeat/filebeat.tpl /opt/log-pilot

WORKDIR /opt/log-pilot
CMD ["/opt/log-pilot/bin/log-pilot"]
