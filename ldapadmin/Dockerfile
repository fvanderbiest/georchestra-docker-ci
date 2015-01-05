#
# Dockerfile for the geOrchestra ldapadmin service
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

ADD https://sdi.georchestra.org/ci/job/georchestra-template/lastSuccessfulBuild/artifact/ldapadmin/target/ldapadmin-template.war /var/lib/tomcat7/webapps/ldapadmin.war

RUN mkdir -p /var/lib/tomcat7/webapps/ldapadmin && \
    unzip -e /var/lib/tomcat7/webapps/ldapadmin.war -d /var/lib/tomcat7/webapps/ldapadmin && \
    chown -R tomcat7:tomcat7 /var/lib/tomcat7/webapps/ldapadmin 

# replace the webapp's own config in order to link with the LDAP + PG hosts
COPY webmvc-config.xml /var/lib/tomcat7/webapps/ldapadmin/WEB-INF/spring/
# Better (TODO) replace values with sed in the original webapp's file


COPY truststore /etc/tomcat7/
RUN echo "JAVA_OPTS=\"\${JAVA_OPTS} -Djavax.net.ssl.trustStore=/etc/tomcat7/truststore -Djavax.net.ssl.trustStorePassword=STOREPASSWORD \"" >> /etc/default/tomcat7

# Expose the port where tomcat is listening
EXPOSE 8281

VOLUME ["/tmp"] # server logs

RUN sed -i 's/8080/8281/' /var/lib/tomcat7/conf/server.xml

RUN touch /tmp/ldapadmin.log && \
    chown -R tomcat7:tomcat7 /tmp/ldapadmin.log

# Start Tomcat server
# The last line (the CMD command) is used to make a fake always-running
# command (the tail command); thus, the Docker container will keep running.
#~ CMD /etc/init.d/tomcat7 start && tail -F /var/log/tomcat7/catalina.out
CMD /etc/init.d/tomcat7 start && tail -F /tmp/ldapadmin.log

