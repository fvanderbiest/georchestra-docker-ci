#
# Dockerfile for the geOrchestra catalogapp service
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

ADD https://sdi.georchestra.org/ci/job/georchestra-template/lastSuccessfulBuild/artifact/catalogapp/target/catalogapp-template.war /var/lib/tomcat8/webapps/catalogapp.war
RUN chmod a+r /var/lib/tomcat8/webapps/*.war

COPY truststore /etc/tomcat8/
RUN echo "JAVA_OPTS=\"\${JAVA_OPTS} -Djavax.net.ssl.trustStore=/etc/tomcat8/truststore -Djavax.net.ssl.trustStorePassword=STOREPASSWORD \"" >> /etc/default/tomcat8

# Expose the port where tomcat is listening
EXPOSE 8282

#VOLUME ["/tmp"] # server logs

RUN sed -i 's/8080/8282/' /var/lib/tomcat8/conf/server.xml

# Start Tomcat server
# The last line (the CMD command) is used to make a fake always-running
# command (the tail command); thus, the Docker container will keep running.
CMD /etc/init.d/tomcat8 start && tail -F /var/log/tomcat8/catalina.out