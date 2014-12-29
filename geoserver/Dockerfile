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

# add contrib repository to fetch "core fonts for the web"
RUN echo "deb http://ftp.fr.debian.org/debian jessie contrib" >> /etc/apt/sources.list
RUN echo "deb http://security.debian.org/ jessie/updates contrib" >> /etc/apt/sources.list

RUN apt-get update && \
    apt-get install -y tomcat7 wget ttf-mscorefonts-installer gdal-bin
    
# gdal-bin required for ogr extension (ogr2ogr)
    
RUN mkdir /opt/geoserver_data_dir /var/cache/geowebcache_cache_dir && \
    chown tomcat7 /opt/geoserver_data_dir /var/cache/geowebcache_cache_dir

# Environment variables
ENV GEOSERVER_DATA_DIR /opt/geoserver_data_dir
# it also has GWC built in: decide whether to keep it or not ...
ENV GEOWEBCACHE_CACHE_DIR /var/cache/geowebcache_cache_dir

# TODO: generate + use keystore
#sudo keytool -genkey -dname "CN=localhost, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown" -alias georchestra_localhost -keystore /etc/tomcat7/keystore -storepass STOREPASSWORD -keypass STOREPASSWORD -keyalg RSA -keysize 2048

# override java opts at run time with eg docker run -d -p 49154:8080 --name geoserver0 -e "JAVA_OPTS=xxxx" geor_geoserver
ENV JAVA_OPTS "-Djava.awt.headless=true \
            -Xms1G -Xmx1G -XX:PermSize=256m -XX:MaxPermSize=256m \
            -Dfile.encoding=UTF8 \
            -Djavax.servlet.request.encoding=UTF-8 \
            -Djavax.servlet.response.encoding=UTF-8 \
            -server \
            -XX:+UseParNewGC -XX:ParallelGCThreads=2 \
            -XX:SoftRefLRUPolicyMSPerMB=36000 \
            -XX:NewRatio=2 \
            -XX:+AggressiveOpts"
            
            #~ -Djavax.net.ssl.trustStore=/etc/tomcat6/keystore \
            #~ -Djavax.net.ssl.trustStorePassword=STOREPASSWORD

# FIXME: this WAR was built with GeoFence, and we don't want this by default:
# FIXME: cannot use ADD because of self-signed certificate for sdi.georchestra.org
RUN wget --no-check-certificate https://sdi.georchestra.org/ci/job/georchestra-template/lastSuccessfulBuild/artifact/geoserver/webapp/target/geoserver-template.war -O /var/lib/tomcat7/webapps/geoserver.war

# Expose the port where tomcat is listening
EXPOSE 8080

# Add VOLUMEs to allow backup & sharing
VOLUME  ["/opt/geoserver_data_dir", "/var/cache/geowebcache_cache_dir"]

# Start Tomcat server
# The last line (the CMD command) is used to make a fake always-running
# command (the tail command); thus, the Docker container will keep running.
#~ CMD sudo /etc/init.d/tomcat7 start && tail -f /var/log/tomcat7/catalina.out
CMD /etc/init.d/tomcat7 start && tail -F /var/log/tomcat7/catalina.out
