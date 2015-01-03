#
# Dockerfile for the geOrchestra apache service
#
# Use with caution: this is work in progress
#

FROM debian:jessie

MAINTAINER PSC "psc@georchestra.org"

ENV DEBIAN_FRONTEND noninterative

RUN apt-get update && \
    apt-get install -y apache2 ca-certificates sed

# Enable apache mods.
RUN a2enmod proxy_connect proxy proxy_http ssl rewrite headers
# TODO: check proxy_connect is required
# TODO: check the use of deflate

# Copy modules configuration files
ADD georchestra-site /var/www/georchestra/
RUN sed -i 's/localhost:8180/proxy_host:8180/' /var/www/georchestra/conf/*.conf 
# Create the apache site
ADD apache-config.conf /etc/apache2/sites-enabled/georchestra.conf

RUN a2dissite 000-default default-ssl

RUN echo "127.0.0.1       vm-georchestra" >> /etc/hosts

# SSL TODO
# see eg http://www.unix.com/shell-programming-and-scripting/107305-shell-script-provide-answers-ssl-cert-request.html

# Expose the web ports
EXPOSE 80 443

# Add VOLUMEs to allow backups
VOLUME ["/tmp"] # server logs

# Set the default command to run when starting the container
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
