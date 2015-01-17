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

RUN apt-get update && \
    apt-get install -y tomcat7 unzip

RUN rm -rf /var/lib/tomcat7/webapps/*

# native libs
RUN apt-get install -y libgdal1h libgdal-java

ADD https://sdi.georchestra.org/ci/job/georchestra-template/lastSuccessfulBuild/artifact/mapfishapp/target/mapfishapp-template.war /var/lib/tomcat7/webapps/mapfishapp.war

RUN mkdir -p /var/lib/tomcat7/webapps/mapfishapp && \
    unzip -e /var/lib/tomcat7/webapps/mapfishapp.war -d /var/lib/tomcat7/webapps/mapfishapp && \
    chown -R tomcat7:tomcat7 /var/lib/tomcat7/webapps/mapfishapp

# replace the webapp's own ws-servlet.xml in order to link with the database
COPY ws-servlet.xml /var/lib/tomcat7/webapps/mapfishapp/WEB-INF/

COPY truststore /etc/tomcat7/
RUN echo "JAVA_OPTS=\"\${JAVA_OPTS} -Djavax.net.ssl.trustStore=/etc/tomcat7/truststore -Djavax.net.ssl.trustStorePassword=STOREPASSWORD \"" >> /etc/default/tomcat7

# GDAL java bindings
RUN rm -rf /var/lib/tomcat7/webapps/mapfishapp/WEB-INF/lib/imageio-ext-gdalframework-1.1.7.jar
RUN cp /usr/share/java/gdal.jar /var/lib/tomcat7/webapps/mapfishapp/WEB-INF/lib/gdal.jar

# Expose the port where tomcat is listening
EXPOSE 8280

VOLUME ["/tmp"] # server logs

RUN sed -i 's/8080/8280/' /var/lib/tomcat7/conf/server.xml

# Start Tomcat server
# The last line (the CMD command) is used to make a fake always-running
# command (the tail command); thus, the Docker container will keep running.
CMD /etc/init.d/tomcat7 start && tail -F /var/log/tomcat7/catalina.out