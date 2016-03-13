#
# Dockerfile for the geOrchestra openldap service
#
# Use with caution: this is work in progress
#

FROM debian:jessie

MAINTAINER PSC "psc@georchestra.org"

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && \
    apt-get install -y slapd ldap-utils wget ca-certificates

# FIXME: find a way to customize the "secret" password
# Hint: environment variables

RUN /etc/init.d/slapd start && \
    wget --no-check-certificate -O - https://raw.githubusercontent.com/georchestra/georchestra/15.12/ldap/bootstrap.ldif | ldapadd -Y EXTERNAL -H ldapi:/// && \
    wget --no-check-certificate -O - https://raw.githubusercontent.com/georchestra/georchestra/15.12/ldap/groupofmembers.ldif | ldapadd -Y EXTERNAL -H ldapi:/// && \
    wget --no-check-certificate -O - https://raw.githubusercontent.com/georchestra/georchestra/15.12/ldap/root.ldif | ldapadd -D"cn=admin,dc=georchestra,dc=org" -w secret && \
    wget --no-check-certificate -O - https://raw.githubusercontent.com/georchestra/georchestra/15.12/ldap/georchestra.ldif | ldapadd -D"cn=admin,dc=georchestra,dc=org" -w secret

# Expose the ldap port
EXPOSE 389

# Add VOLUMEs to allow backup of databases
#~ VOLUME  ["/etc/ldap/slapd.d"]

# Set the default command to run when starting the container
CMD /usr/sbin/slapd -h "ldap:/// ldapi:///" -g openldap -u openldap -F /etc/ldap/slapd.d -d 256
