FROM perconalab/percona-postgres-operator:main-postgres
# crunchy-postgres image runs as USER 26, switch back to install packages
USER 0

# ===== Early lines ordered for leveraging cache, reorder carefully =====
# For RHEL8 all arguments used in main code has to be specified after FROM
ENV PG_MAJOR 13

# Preserving PGVERSION out of paranoia
ENV PGROOT="/usr/pgsql-${PG_MAJOR}" PGVERSION="${PG_MAJOR}"

RUN set -ex; \
    curl -Lf -o /tmp/python3-psutil.rpm http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/python3-psutil-5.4.3-10.el8.x86_64.rpm; \
    curl -Lf -o /tmp/python3-prettytable.rpm  http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/python3-prettytable-0.7.2-14.el8.noarch.rpm; \
    rpmkeys --checksig /tmp/python3-prettytable.rpm /tmp/python3-psutil.rpm; \
    rpm -i /tmp/python3-prettytable.rpm /tmp/python3-psutil.rpm; \
    rm -rf /tmp/python3-prettytable.rpm /tmp/python3-psutil.rpm

RUN set -ex; \
    microdnf -y install \
		--enablerepo="epel" \
		python3-pip \
        tzdata \
		python3-psycopg2 \
        percona-patroni; \
	microdnf -y clean all --enablerepo="epel"

ENV PATH="${PGROOT}/bin:${PATH}"

RUN mkdir -p /tablespaces

# Adjust ownership for the folders to be the "postgres" user and allow the group
# permissions to match the user ones EXCEPT for the /tablespaces folder, which
# will only have permissions on the user
RUN chown -R postgres:postgres /tablespaces; \
	chmod -R g=u /tablespaces

# open up the postgres port
EXPOSE 5432

COPY bin/postgres-ha /opt/crunchy/bin/postgres-ha
COPY conf/postgres-ha /opt/crunchy/conf/postgres-ha

RUN set -ex; \
    curl -Lf -o /opt/crunchy/bin/yq  https://github.com/mikefarah/yq/releases/download/3.3.2/yq_linux_amd64; \
    chmod +x /opt/crunchy/bin/yq

# The VOLUME directive must appear after all RUN directives to ensure the proper
# volume permissions are applied when building the image
VOLUME ["/pgdata", "/pgwal", "/pgconf", "/backrestrepo", "/sshd"]

ENTRYPOINT ["/opt/crunchy/bin/postgres-ha/bootstrap-postgres-ha.sh"]

USER 26

CMD ["/opt/patroni/bin/patroni"]
