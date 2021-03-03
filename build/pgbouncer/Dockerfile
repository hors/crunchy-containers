FROM perconalab/percona-postgres-operator:main-base

RUN set -ex; \
    curl -Lf -o /tmp/c-ares.rpm http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/c-ares-1.13.0-5.el8.x86_64.rpm; \
    rpmkeys --checksig /tmp/c-ares.rpm; \
    rpm -i /tmp/c-ares.rpm; \
    rm -rf /tmp/c-ares.rpm

RUN set -ex; \
    microdnf -y install \
	--disablerepo="epel" \
    c-ares \
	percona-pgbouncer; \
	microdnf -y clean all

ENV PGVERSION 13

RUN mkdir -p /opt/crunchy/bin /opt/crunchy/conf /pgconf

COPY bin/pgbouncer /opt/crunchy/bin
COPY bin/common /opt/crunchy/bin
COPY conf/pgbouncer /opt/crunchy/conf

RUN chown -R 2:0 /opt/crunchy /pgconf; \
	chmod -R g=u /opt/crunchy /pgconf

EXPOSE 6432

RUN chmod g=u /etc/passwd

# The VOLUME directive must appear after all RUN directives to ensure the proper
# volume permissions are applied when building the image
VOLUME ["/pgconf"]

ENTRYPOINT ["opt/crunchy/bin/uid_daemon.sh"]

USER 2

CMD ["/opt/crunchy/bin/start.sh"]
