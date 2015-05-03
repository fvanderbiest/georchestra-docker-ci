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
    apt-get install -y tomcat7 unzip

RUN rm -rf /var/lib/tomcat7/webapps/*

ADD https://sdi.georchestra.org/ci/job/georchestra-template/lastSuccessfulBuild/artifact/security-proxy/target/ROOT-template.war /var/lib/tomcat7/webapps/ROOT.war
ADD https://sdi.georchestra.org/ci/job/georchestra-template/lastSuccessfulBuild/artifact/cas-server-webapp/target/cas-template.war /var/lib/tomcat7/webapps/cas.war
ADD https://sdi.georchestra.org/ci/job/georchestra-template/lastSuccessfulBuild/artifact/header/target/header-template.war /var/lib/tomcat7/webapps/header-private.war
RUN chmod a+r /var/lib/tomcat7/webapps/*.war

RUN mkdir -p /var/lib/tomcat7/webapps/ROOT && \
    unzip -e /var/lib/tomcat7/webapps/ROOT.war -d /var/lib/tomcat7/webapps/ROOT && \
    chown -R tomcat7:tomcat7 /var/lib/tomcat7/webapps/ROOT

RUN mkdir -p /var/lib/tomcat7/webapps/cas && \
    unzip -e /var/lib/tomcat7/webapps/cas.war -d /var/lib/tomcat7/webapps/cas && \
    chown -R tomcat7:tomcat7 /var/lib/tomcat7/webapps/cas

# replace the webapp's own proxy-servlet.xml & properties file in order to link with the database and other containers
COPY proxy-servlet.xml /var/lib/tomcat7/webapps/ROOT/WEB-INF/
COPY security-proxy.properties /var/lib/tomcat7/webapps/ROOT/WEB-INF/
COPY log4j.properties /var/lib/tomcat7/webapps/ROOT/WEB-INF/classes/
# cas customization:
COPY cas.properties /var/lib/tomcat7/webapps/cas/WEB-INF/

# the keystore is for the server (cas)
COPY keystore /etc/tomcat7/
# the trustStore is for the client (cas-client inside proxy)
COPY truststore /etc/tomcat7/
# see http://stackoverflow.com/questions/2200176/keytool-create-a-trusted-self-signed-certificate

RUN echo "JAVA_OPTS=\"\${JAVA_OPTS} -Djavax.net.ssl.trustStore=/etc/tomcat7/truststore -Djavax.net.ssl.trustStorePassword=STOREPASSWORD -Xms256m -Xmx256m \"" >> /etc/default/tomcat7

## need proxycas host to know his name as relayed by apache
# FIXME: does not work for some reason ...
#RUN echo "127.0.0.1       proxycas" >> /etc/hosts

ADD https://github.com/hiteshjasani/nim-mungehosts/releases/download/v0.1.1/mungehosts /usr/local/bin/mungehosts
RUN chmod 755 /usr/local/bin/mungehosts

# configure connector on port 8443 for CAS ticket validation:
COPY server.xml /var/lib/tomcat7/conf/

# Expose the port where tomcat is listening
#RUN sed -i 's/8080/8180/' /var/lib/tomcat7/conf/server.xml
# line above is useless since the port has already been customized in server.xml
EXPOSE 8180

VOLUME ["/tmp"] # server logs

# Start Tomcat server
# The last line (the CMD command) is used to make a fake always-running
# command (the tail command); thus, the Docker container will keep running.
CMD mungehosts -l proxycas && /etc/init.d/tomcat7 start && tail -F /var/log/tomcat7/catalina.out

