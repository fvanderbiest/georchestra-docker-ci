#
# Dockerfile for the geOrchestra geonetwork service
#
# Use with caution: this is work in progress
#
# run this one with --privileged to overcome https://github.com/docker/docker/issues/6800
# (docker run -it --cap-add SYS_PTRACE does not fix it)

FROM debian:jessie

MAINTAINER PSC "psc@georchestra.org"

ENV DEBIAN_FRONTEND noninterative

RUN apt-get update && \
    apt-get install -y tomcat8 unzip

RUN rm -rf /var/lib/tomcat8/webapps/*

ADD https://sdi.georchestra.org/ci/job/georchestra-template/lastSuccessfulBuild/artifact/geonetwork/web/target/geonetwork-template.war /var/lib/tomcat8/webapps/geonetwork.war

RUN mkdir -p /var/lib/tomcat8/webapps/geonetwork && \
    unzip -e /var/lib/tomcat8/webapps/geonetwork.war -d /var/lib/tomcat8/webapps/geonetwork && \
    chown -R tomcat8:tomcat8 /var/lib/tomcat8/webapps/geonetwork

# replace the webapp's own config
COPY config-security.properties /var/lib/tomcat8/webapps/geonetwork/WEB-INF/
COPY config-overrides-georchestra.xml /var/lib/tomcat8/webapps/geonetwork/WEB-INF/

COPY truststore /etc/tomcat8/
RUN echo "JAVA_OPTS=\"\${JAVA_OPTS} -Djavax.net.ssl.trustStore=/etc/tomcat8/truststore -Djavax.net.ssl.trustStorePassword=STOREPASSWORD \"" >> /etc/default/tomcat8

# add the GN data dir submodule as a submodule
RUN mkdir /opt/geonetwork_data_dir
COPY geonetwork_minimal_datadir /opt/geonetwork_data_dir
RUN chown -R tomcat8 /opt/geonetwork_data_dir

# TODO: add locally the GN submodule as a submodule
# then copy it into the VM
RUN echo "JAVA_OPTS=\"\${JAVA_OPTS} -Dgeonetwork.dir=/opt/geonetwork_data_dir -Dgeonetwork.jeeves.configuration.overrides.file=/var/lib/tomcat8/webapps/geonetwork/WEB-INF/config-overrides-georchestra.xml \"" >> /etc/default/tomcat8

# workaround for https://github.com/georchestra/georchestra/issues/871
RUN echo "JAVA_OPTS=\"\${JAVA_OPTS} -Djava.util.prefs.userRoot=/tmp -Djava.util.prefs.systemRoot=/tmp \"" >> /etc/default/tomcat8

# Expose the port where tomcat is listening
RUN sed -i 's/8080/8287/' /var/lib/tomcat8/conf/server.xml
EXPOSE 8287

VOLUME ["/tmp"] # server logs

# Start Tomcat server
# The last line (the CMD command) is used to make a fake always-running
# command (the tail command); thus, the Docker container will keep running.
CMD /etc/init.d/tomcat8 start && tail -F /var/log/tomcat8/catalina.out