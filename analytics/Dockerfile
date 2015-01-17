#
# Dockerfile for the geOrchestra analytics service
#
# Use with caution: this is work in progress
#
# run this one with --privileged to overcome https://github.com/docker/docker/issues/6800
# (docker run -it --cap-add SYS_PTRACE does not fix it)

FROM debian:jessie

MAINTAINER PSC "psc@georchestra.org"

ENV DEBIAN_FRONTEND noninterative

RUN apt-get update && \
    apt-get install -y tomcat7 unzip

RUN rm -rf /var/lib/tomcat7/webapps/*

ADD https://sdi.georchestra.org/ci/job/georchestra-template/lastSuccessfulBuild/artifact/analytics/target/analytics-template.war /var/lib/tomcat7/webapps/analytics.war

RUN mkdir -p /var/lib/tomcat7/webapps/analytics && \
    unzip -e /var/lib/tomcat7/webapps/analytics.war -d /var/lib/tomcat7/webapps/analytics && \
    chown -R tomcat7:tomcat7 /var/lib/tomcat7/webapps/analytics

# replace the webapp's own config
COPY analytics.properties /var/lib/tomcat7/webapps/analytics/WEB-INF/
COPY config.js /var/lib/tomcat7/webapps/analytics/js/app/

COPY truststore /etc/tomcat7/
RUN echo "JAVA_OPTS=\"\${JAVA_OPTS} -Djavax.net.ssl.trustStore=/etc/tomcat7/truststore -Djavax.net.ssl.trustStorePassword=STOREPASSWORD \"" >> /etc/default/tomcat7

# Expose the port where tomcat is listening
EXPOSE 8286

VOLUME ["/tmp"] # server logs

RUN sed -i 's/8080/8286/' /var/lib/tomcat7/conf/server.xml

# Start Tomcat server
# The last line (the CMD command) is used to make a fake always-running
# command (the tail command); thus, the Docker container will keep running.
CMD /etc/init.d/tomcat7 start && tail -F /var/log/tomcat7/catalina.out