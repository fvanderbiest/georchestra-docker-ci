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

RUN echo "deb http://build.georchestra.org/debian jessie main" >> /etc/apt/sources.list
ADD build.georchestra.org.asc /tmp/build.georchestra.org.asc
RUN apt-key add /tmp/build.georchestra.org.asc

RUN apt-get update && \
    apt-get install -y tomcat8 unzip

RUN rm -rf /var/lib/tomcat8/webapps/*

# install datadir:
ADD https://github.com/georchestra/datadir/archive/docker-15.12.zip /tmp/georchestra.zip
RUN unzip /tmp/georchestra.zip -d /etc && mv /etc/datadir-docker-15.12 /etc/georchestra

RUN apt-get install -y georchestra-security-proxy georchestra-cas georchestra-header

RUN ln -s /usr/share/lib/georchestra-security-proxy/ROOT-generic.war /var/lib/tomcat8/webapps/ROOT.war
RUN ln -s /usr/share/lib/georchestra-cas/cas-generic.war /var/lib/tomcat8/webapps/cas.war
RUN ln -s /usr/share/lib/georchestra-header/header-generic.war /var/lib/tomcat8/webapps/header-private.war

# unzip webapps to allow overrides via overlay
RUN mkdir -p /var/lib/tomcat8/webapps/ROOT && \
    unzip -e /var/lib/tomcat8/webapps/ROOT.war -d /var/lib/tomcat8/webapps/ROOT && \
    chown -R tomcat8:tomcat8 /var/lib/tomcat8/webapps/ROOT
RUN mkdir -p /var/lib/tomcat8/webapps/cas && \
    unzip -e /var/lib/tomcat8/webapps/cas.war -d /var/lib/tomcat8/webapps/cas && \
    chown -R tomcat8:tomcat8 /var/lib/tomcat8/webapps/cas
RUN mkdir -p /var/lib/tomcat8/webapps/header-private && \
    unzip -e /var/lib/tomcat8/webapps/header-private.war -d /var/lib/tomcat8/webapps/header-private && \
    chown -R tomcat8:tomcat8 /var/lib/tomcat8/webapps/header-private

# the keystore is for the server (cas)
COPY keystore /etc/tomcat8/
# the trustStore is for the client (cas-client inside proxy)
COPY truststore /etc/tomcat8/
# see http://stackoverflow.com/questions/2200176/keytool-create-a-trusted-self-signed-certificate

RUN echo "JAVA_OPTS=\"\${JAVA_OPTS} -Djavax.net.ssl.trustStore=/etc/tomcat8/truststore -Djavax.net.ssl.trustStorePassword=STOREPASSWORD -Xms256m -Xmx256m \"" >> /etc/default/tomcat8

RUN echo "JAVA_OPTS=\"\${JAVA_OPTS} -Dgeorchestra.datadir=/etc/georchestra \"" >> /etc/default/tomcat8

## need proxycas host to know his name as relayed by apache
# FIXME: does not work for some reason ...
#RUN echo "127.0.0.1       proxycas" >> /etc/hosts

ADD https://github.com/hiteshjasani/nim-mungehosts/releases/download/v0.1.1/mungehosts /usr/local/bin/mungehosts
RUN chmod 755 /usr/local/bin/mungehosts

# configure connector on port 8443 for CAS ticket validation:
# Expose the port where tomcat is listening
RUN sed -i 's/8080/8180/' /var/lib/tomcat8/conf/server.xml
EXPOSE 8180

VOLUME ["/tmp"]

# Start Tomcat server
# The last line (the CMD command) is used to make a fake always-running
# command (the tail command); thus, the Docker container will keep running.
CMD mungehosts -l proxycas && /etc/init.d/tomcat8 start && tail -F /var/log/tomcat8/catalina.out
