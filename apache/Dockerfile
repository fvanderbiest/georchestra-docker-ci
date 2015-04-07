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
RUN a2enmod proxy proxy_http ssl rewrite headers deflate

# Copy modules configuration files
ADD georchestra-site /var/www/georchestra/
RUN sed -i 's/localhost:8180/proxycas:8180/' /var/www/georchestra/conf/*.conf 
# Create the apache site
ADD apache-config.conf /etc/apache2/sites-enabled/georchestra.conf

RUN a2dissite 000-default default-ssl

RUN echo "127.0.0.1       georchestra.mydomain.org" >> /etc/hosts

# Expose the web ports
EXPOSE 80 443

# Add VOLUMEs to allow backups
VOLUME ["/tmp"] # server logs

# Set the default command to run when starting the container
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]