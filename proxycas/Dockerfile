#
# Dockerfile for the geOrchestra proxy + cas + header services
#
# Use with caution: this is work in progress
#
# run this one with --privileged to overcome https://github.com/docker/docker/issues/6800
# (docker run -it --cap-add SYS_PTRACE does not fix it)

FROM debian:jessie

MAINTAINER PSC "psc@georchestra.org"

ENV DEBIAN_FRONTEND noninterative

RUN apt-get update && \
    apt-get install -y tomcat6 unzip

RUN rm -rf /var/lib/tomcat6/webapps/*
ADD https://sdi.georchestra.org/ci/job/georchestra-template/lastSuccessfulBuild/artifact/security-proxy/target/ROOT-template.war /var/lib/tomcat6/webapps/ROOT.war
ADD https://sdi.georchestra.org/ci/job/georchestra-template/lastSuccessfulBuild/artifact/cas-server-webapp/target/cas-template.war /var/lib/tomcat6/webapps/cas.war
ADD https://sdi.georchestra.org/ci/job/georchestra-template/lastSuccessfulBuild/artifact/header/target/header-template.war /var/lib/tomcat6/webapps/header-private.war
RUN chmod a+r /var/lib/tomcat6/webapps/*.war

RUN mkdir -p /var/lib/tomcat6/webapps/ROOT && \
    unzip -e /var/lib/tomcat6/webapps/ROOT.war -d /var/lib/tomcat6/webapps/ROOT && \
    chown -R tomcat6:tomcat6 /var/lib/tomcat6/webapps/ROOT

RUN mkdir -p /var/lib/tomcat6/webapps/cas && \
    unzip -e /var/lib/tomcat6/webapps/cas.war -d /var/lib/tomcat6/webapps/cas && \
    chown -R tomcat6:tomcat6 /var/lib/tomcat6/webapps/cas

# replace the webapp's own proxy-servlet.xml & properties file in order to link with the database and other containers
COPY proxy-servlet.xml /var/lib/tomcat6/webapps/ROOT/WEB-INF/
COPY security-proxy.properties /var/lib/tomcat6/webapps/ROOT/WEB-INF/
COPY log4j.properties /var/lib/tomcat6/webapps/ROOT/WEB-INF/classes/
# cas customization:
COPY cas.properties /var/lib/tomcat6/webapps/cas/WEB-INF/

COPY keystore /etc/tomcat6/
ENV JAVA_OPTS "-Djavax.net.ssl.trustStore=/etc/tomcat6/keystore -Djavax.net.ssl.trustStorePassword=STOREPASSWORD"              

RUN sed -i 's/8080/8180/' /var/lib/tomcat6/conf/server.xml

# Expose the port where tomcat is listening
EXPOSE 8180

VOLUME ["/tmp"] # server logs

# Start Tomcat server
# The last line (the CMD command) is used to make a fake always-running
# command (the tail command); thus, the Docker container will keep running.
CMD /etc/init.d/tomcat6 start && tail -F /var/log/tomcat6/catalina.out

