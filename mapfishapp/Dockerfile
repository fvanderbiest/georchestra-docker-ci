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
    apt-get install -y tomcat7 wget

# FIXME: cannot use ADD because of self-signed certificate for sdi.georchestra.org
RUN wget --no-check-certificate https://sdi.georchestra.org/ci/job/georchestra-template/lastSuccessfulBuild/artifact/mapfishapp/target/mapfishapp-template.war -O /var/lib/tomcat7/webapps/mapfishapp.war

# Expose the port where tomcat is listening
EXPOSE 8080

VOLUME ["/tmp"] # server logs

# Start Tomcat server
# The last line (the CMD command) is used to make a fake always-running
# command (the tail command); thus, the Docker container will keep running.
CMD /etc/init.d/tomcat7 start && tail -F /var/log/tomcat7/catalina.out

# replace the webapp's own ws-servlet.xml in order to link with the database
COPY ws-servlet.xml /var/lib/tomcat7/webapps/mapfishapp/WEB-INF/