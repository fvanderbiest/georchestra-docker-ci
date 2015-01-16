#
# Dockerfile for the geOrchestra geoserver service
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

# add contrib repository to fetch "core fonts for the web"
# add non-free for imageio
RUN echo "deb http://ftp.fr.debian.org/debian jessie contrib non-free" >> /etc/apt/sources.list
RUN echo "deb http://security.debian.org/ jessie/updates contrib non-free" >> /etc/apt/sources.list

RUN apt-get update && \
    apt-get install -y libjai-core-java libjai-imageio-core-java
# gdal-bin required for ogr extension (ogr2ogr)

# jai + imageio:
RUN ln -s /usr/share/java/jai_core.jar /usr/share/tomcat7/lib/jai_core.jar
RUN ln -s /usr/share/java/jai_codec.jar /usr/share/tomcat7/lib/jai_codec.jar
RUN ln -s /usr/share/java/mlibwrapper_jai.jar /usr/share/tomcat7/lib/mlibwrapper_jai.jar
RUN ln -s /usr/share/java/jai_imageio.jar /usr/share/tomcat7/lib/jai_imageio.jar
RUN ln -s /usr/share/java/clibwrapper_jiio.jar /usr/share/tomcat7/lib/clibwrapper_jiio.jar

RUN apt-get install -y ttf-mscorefonts-installer libgdal1h libgdal-java gdal-bin

# turbojpeg
ADD http://downloads.sourceforge.net/project/libjpeg-turbo/1.4.0/libjpeg-turbo-official_1.4.0_amd64.deb /tmp/libjpeg-turbo-official_1.4.0_amd64.deb
RUN dpkg -i /tmp/libjpeg-turbo-official_1.4.0_amd64.deb
# TODO: utiliser le package debian officiel

# add the geoserver data dir submodule as a submodule
RUN mkdir /opt/geoserver_data_dir /var/cache/geowebcache_cache_dir
COPY geoserver_minimal_datadir /opt/geoserver_data_dir
RUN chown -R tomcat7 /opt/geoserver_data_dir /var/cache/geowebcache_cache_dir

# then copy it into the VM
COPY truststore /etc/tomcat7/
RUN echo "JAVA_OPTS=\"\${JAVA_OPTS} -Djavax.net.ssl.trustStore=/etc/tomcat7/truststore -Djavax.net.ssl.trustStorePassword=STOREPASSWORD \"" >> /etc/default/tomcat7
# data dirs:
RUN echo "JAVA_OPTS=\"\${JAVA_OPTS} -DGEOSERVER_DATA_DIR=/opt/geoserver_data_dir -DGEOWEBCACHE_CACHE_DIR=/var/cache/geowebcache_cache_dir \"" >> /etc/default/tomcat7
# UTF-8
RUN echo "JAVA_OPTS=\"\${JAVA_OPTS} -Dfile.encoding=UTF8 -Djavax.servlet.request.encoding=UTF-8 -Djavax.servlet.response.encoding=UTF-8 \"" >> /etc/default/tomcat7
# RAM:
RUN sed -i '/java.awt.headless/d' /etc/default/tomcat7
RUN echo "JAVA_OPTS=\"\${JAVA_OPTS} -Xms1G -Xmx1G -XX:PermSize=256m -XX:MaxPermSize=256m \"" >> /etc/default/tomcat7
# optimisations:
RUN echo "JAVA_OPTS=\"\${JAVA_OPTS} -server -Djava.awt.headless=true -XX:+UseParNewGC -XX:ParallelGCThreads=2 -XX:SoftRefLRUPolicyMSPerMB=36000 -XX:NewRatio=2 -XX:+AggressiveOpts \"" >> /etc/default/tomcat7
# turboJPEG:
# gdal jni:
RUN echo "JAVA_OPTS=\"\${JAVA_OPTS} -Djava.library.path=/usr/lib/jni:/opt/libjpeg-turbo/lib64/\"" >> /etc/default/tomcat7

ADD https://sdi.georchestra.org/ci/job/georchestra-template/lastSuccessfulBuild/artifact/geoserver/webapp/target/geoserver-template.war /var/lib/tomcat7/webapps/geoserver.war

RUN mkdir -p /var/lib/tomcat7/webapps/geoserver && \
    unzip -e /var/lib/tomcat7/webapps/geoserver.war -d /var/lib/tomcat7/webapps/geoserver && \
    chown -R tomcat7:tomcat7 /var/lib/tomcat7/webapps/geoserver

# GDAL java bindings
RUN rm -rf /var/lib/tomcat7/webapps/geoserver/WEB-INF/lib/imageio-ext-gdal-bindings-1.9.2.jar
RUN cp /usr/share/java/gdal.jar /var/lib/tomcat7/webapps/geoserver/WEB-INF/lib/gdal.jar

# Expose the port where tomcat is listening
RUN sed -i 's/8080/8380/' /var/lib/tomcat7/conf/server.xml
EXPOSE 8380

# Create volume for geoserver data (shapefiles, etc)
RUN mkdir /data && chown tomcat7 /data

VOLUME  ["/tmp", "/data"]

# Start Tomcat server
# The last line (the CMD command) is used to make a fake always-running
# command (the tail command); thus, the Docker container will keep running.
CMD /etc/init.d/tomcat7 start && tail -F /var/log/tomcat7/catalina.out
