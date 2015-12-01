#
# Dockerfile for the geOrchestra viewer service
#
# Use with caution: this is work in progress
#
# run this one with --privileged to overcome https://github.com/docker/docker/issues/6800
# (docker run -it --cap-add SYS_PTRACE does not fix it)

FROM debian:jessie

MAINTAINER PSC "psc@georchestra.org"

ENV DEBIAN_FRONTEND noninterative

RUN echo "deb http://build.georchestra.org/debian jessie main" >> /etc/apt/sources.list
ADD http://build.georchestra.org/debian/build.georchestra.org.asc /tmp/build.georchestra.org.asc
RUN apt-key add /tmp/build.georchestra.org.asc

RUN apt-get update && \
    apt-get install -y tomcat8

RUN rm -rf /var/lib/tomcat8/webapps/*

# native libs
RUN apt-get install -y libgdal1h libgdal-java

RUN apt-get install -y georchestra-mapfishapp

RUN ln -s /usr/share/lib/georchestra-mapfishapp/mapfishapp-generic.war /var/lib/tomcat8/webapps/mapfishapp.war

COPY truststore /etc/tomcat8/
RUN echo "JAVA_OPTS=\"\${JAVA_OPTS} -Djavax.net.ssl.trustStore=/etc/tomcat8/truststore -Djavax.net.ssl.trustStorePassword=STOREPASSWORD \"" >> /etc/default/tomcat8
RUN echo "JAVA_OPTS=\"\${JAVA_OPTS} -Dgeorchestra.datadir=/etc/georchestra \"" >> /etc/default/tomcat8

# GDAL java bindings
#~ RUN rm -rf /var/lib/tomcat8/webapps/mapfishapp/WEB-INF/lib/imageio-ext-gdalframework-1.1.7.jar
#~ RUN cp /usr/share/java/gdal.jar /var/lib/tomcat8/webapps/mapfishapp/WEB-INF/lib/gdal.jar

# Expose the port where tomcat is listening
EXPOSE 8280

VOLUME ["/tmp"] # server logs

RUN sed -i 's/8080/8280/' /var/lib/tomcat8/conf/server.xml

# Start Tomcat server
# The last line (the CMD command) is used to make a fake always-running
# command (the tail command); thus, the Docker container will keep running.
CMD /etc/init.d/tomcat8 start && tail -F /var/log/tomcat8/catalina.out