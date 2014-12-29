#
# Dockerfile for the geOrchestra apache service
#
# Use with caution: this is work in progress
#

FROM debian:jessie

MAINTAINER PSC "psc@georchestra.org"

ENV DEBIAN_FRONTEND noninterative

RUN apt-get update && \
    apt-get install -y apache2 ca-certificates

# Enable apache mods.
RUN a2enmod proxy_connect proxy_http proxy ssl rewrite headers

# Create directory structure
RUN mkdir -p /var/www/georchestra/htdocs && \
    mkdir /var/www/georchestra/conf && \
    mkdir /var/www/georchestra/logs && \
    mkdir /var/www/georchestra/ssl && \
    chgrp www-data /var/www/georchestra/logs/ && \
    chmod g+w /var/www/georchestra/logs/

RUN a2dissite 000-default default-ssl

# Create the apache site
ADD apache-config.conf /etc/apache2/sites-enabled/georchestra.conf

RUN echo "127.0.0.1       vm-georchestra" >> /etc/hosts

# Copy modules configuration files
ADD conf /var/www/georchestra/conf/

# SSL TODO
# see eg http://www.unix.com/shell-programming-and-scripting/107305-shell-script-provide-answers-ssl-cert-request.html
ADD ssl /var/www/georchestra/ssl/

# Expose the web ports
EXPOSE 80 443

# Add VOLUMEs to allow backups
VOLUME ["/var/www/georchestra"]

# Set the default command to run when starting the container
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
