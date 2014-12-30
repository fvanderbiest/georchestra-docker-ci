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
    apt-get install -y tomcat6 wget

# FIXME: cannot use ADD because of self-signed certificate for sdi.georchestra.org
RUN wget --no-check-certificate https://sdi.georchestra.org/ci/job/georchestra-template/lastSuccessfulBuild/artifact/security-proxy/target/ROOT-template.war -O /var/lib/tomcat6/webapps/ROOT.war
RUN wget --no-check-certificate https://sdi.georchestra.org/ci/job/georchestra-template/lastSuccessfulBuild/artifact/cas-server-webapp/target/cas-template.war -O /var/lib/tomcat6/webapps/cas.war
RUN wget --no-check-certificate https://sdi.georchestra.org/ci/job/georchestra-template/lastSuccessfulBuild/artifact/header/target/header-template.war -O /var/lib/tomcat6/webapps/header-private.war

# Expose the port where tomcat is listening
EXPOSE 8080

VOLUME ["/tmp"] # server logs

# Start Tomcat server
# The last line (the CMD command) is used to make a fake always-running
# command (the tail command); thus, the Docker container will keep running.
CMD /etc/init.d/tomcat6 start && tail -F /var/log/tomcat6/catalina.out

# replace the webapp's own proxy-servlet.xml & properties file in order to link with the database and other containers
COPY proxy-servlet.xml /var/lib/tomcat6/webapps/ROOT/WEB-INF/
COPY security-proxy.properties /var/lib/tomcat6/webapps/ROOT/WEB-INF/