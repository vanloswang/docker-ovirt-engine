# TODO: log setup log and boot.log to stdout only
# TODO: fix gpg key warnings
# TODO: make sure unicode is used
# TODO: give ovirt user a well known gid and uid
# TODO: fix redirect from http to https (port is missing)

FROM centos:7.2.1511

MAINTAINER Roman Mohr <rmohr@redhat.com>

# Database
ENV POSTGRES_USER engine
ENV POSTGRES_PASSWORD engine
ENV POSTGRES_DB engine
ENV POSTGRES_HOST postgres
ENV POSTGRES_PORT 5432

# oVirt
ENV OVIRT_FQDN localhost

EXPOSE 8080 8443

USER root

RUN yum -y localinstall http://resources.ovirt.org/pub/yum-repo/ovirt-release40.rpm \
    && yum -y install ovirt-engine patch \
    && yum -y clean all
    
# engine-setup needs the link to initctl
RUN ln -s /usr/sbin/service /usr/bin/initctl

#oVirt
ENV OVIRT_PASSWORD engine
ENV OVIRT_PKI_ORGANIZATION oVirt

COPY entrypoint.sh answers.conf.in setup.patch /
COPY 999-ovirt-engine.conf /etc/ovirt-engine/engine.conf.d/

# patch engine-setup so that it does not try to start or stop services with systemd
RUN patch -p0 < setup.patch

# For kubernetes copy pki template files to a backup directory
RUN cp -a /etc/pki/ovirt-engine /etc/pki/ovirt-engine.tmpl

# Persist this folder to keep the generated TLS certificates on the first start
VOLUME /etc/pki/ovirt-engine
# Persist this folder to keep the database backups
VOLUME /var/lib/ovirt-engine/backups

# Encrypt host communication
ENV HOST_ENCRYPT=true
# Try to provision hosts when they are added
ENV HOST_INSTALL=true

CMD bash /entrypoint.sh
