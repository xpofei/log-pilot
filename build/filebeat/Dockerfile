FROM cargo.caicloud.xyz/library/filebeat:6.5.4

USER root
RUN mkdir /etc/filebeat
COPY build/filebeat/k8s-log-template.json /etc/filebeat/k8s-log-template.json
COPY build/filebeat/filebeat.yml.tpl /etc/filebeat/filebeat.yml.tpl
COPY bin/filebeat-keeper /opt/filebeat-keeper/filebeat-keeper

WORKDIR /usr/share/filebeat
CMD ["/opt/filebeat-keeper/filebeat-keeper"]
