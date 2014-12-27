#
# Dockerfile for the geOrchestra openldap service
#
# Use with caution: this is work in progress
#

FROM debian:wheezy
# TODO: use Jessie, see https://github.com/georchestra/georchestra/issues/856

MAINTAINER PSC "psc@georchestra.org"

ENV DEBIAN_FRONTEND noninterative

RUN apt-get update && \
    apt-get install -y slapd ldap-utils wget

# FIXME: find a way to customize the "secret" password
RUN /etc/init.d/slapd start && \
    wget --no-check-certificate -O - https://raw.githubusercontent.com/georchestra/LDAP/master/georchestra-bootstrap.ldif | ldapadd -Y EXTERNAL -H ldapi:/// && \
    wget --no-check-certificate -O - https://raw.githubusercontent.com/georchestra/LDAP/master/georchestra-root.ldif | ldapadd -D"cn=admin,dc=georchestra,dc=org" -w secret && \
    wget --no-check-certificate -O - https://raw.githubusercontent.com/georchestra/LDAP/master/georchestra.ldif | ldapadd -D"cn=admin,dc=georchestra,dc=org" -w secret

# Expose the PostgreSQL port
EXPOSE 389

# Add VOLUMEs to allow backup of databases
VOLUME  ["/etc/ldap/slapd.d"]

# Set the default command to run when starting the container
CMD ["/usr/sbin/slapd", "-h", "ldap:///", "ldapi:///", "-g", "openldap", "-u", "openldap", "-F", "/etc/ldap/slapd.d"]
