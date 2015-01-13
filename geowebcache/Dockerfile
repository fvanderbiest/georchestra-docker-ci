#
# Dockerfile for the geOrchestra tile cache service
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

# add non-free for imageio
RUN echo "deb http://ftp.fr.debian.org/debian jessie contrib non-free" >> /etc/apt/sources.list
RUN echo "deb http://security.debian.org/ jessie/updates contrib non-free" >> /etc/apt/sources.list

RUN apt-get install -y libjai-core-java libjai-imageio-core-java

# jai + imageio:
RUN ln -s /usr/share/java/jai_core.jar /usr/share/tomcat7/lib/jai_core.jar
RUN ln -s /usr/share/java/jai_codec.jar /usr/share/tomcat7/lib/jai_codec.jar
RUN ln -s /usr/share/java/mlibwrapper_jai.jar /usr/share/tomcat7/lib/mlibwrapper_jai.jar
RUN ln -s /usr/share/java/jai_imageio.jar /usr/share/tomcat7/lib/jai_imageio.jar
RUN ln -s /usr/share/java/clibwrapper_jiio.jar /usr/share/tomcat7/lib/clibwrapper_jiio.jar

ADD https://sdi.georchestra.org/ci/job/georchestra-template/lastSuccessfulBuild/artifact/geowebcache-webapp/target/geowebcache-template.war /var/lib/tomcat7/webapps/geowebcache.war

COPY truststore /etc/tomcat7/
RUN echo "JAVA_OPTS=\"\${JAVA_OPTS} -Djavax.net.ssl.trustStore=/etc/tomcat7/truststore -Djavax.net.ssl.trustStorePassword=STOREPASSWORD \"" >> /etc/default/tomcat7

# Expose the port where tomcat is listening
EXPOSE 8284

VOLUME ["/tmp"] # server logs

RUN sed -i 's/8080/8284/' /var/lib/tomcat7/conf/server.xml

# Start Tomcat server
# The last line (the CMD command) is used to make a fake always-running
# command (the tail command); thus, the Docker container will keep running.
CMD /etc/init.d/tomcat7 start && tail -F /var/log/tomcat7/catalina.out