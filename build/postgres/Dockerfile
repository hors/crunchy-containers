FROM perconalab/percona-postgres-operator:main-base


# ===== Early lines ordered for leveraging cache, reorder carefully =====
ENV PG_MAJOR 13

RUN set -ex; \
    microdnf -y install \
        libpq \
        llvm9.0 \
        openssh \
	    percona-postgresql${PG_MAJOR//.}; \
    microdnf -y clean all

# Preserving PGVERSION out of paranoia
ENV PGROOT="/usr/pgsql-${PG_MAJOR}" PGVERSION="${PG_MAJOR}"

RUN set -ex; \
    curl -Lf -o /tmp/perl-DBI.rpm http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/perl-DBI-1.641-3.module_el8.1.0+199+8f0a6bbd.x86_64.rpm; \
    curl -Lf -o /tmp/openssh-server.rpm http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/openssh-server-8.0p1-5.el8.x86_64.rpm; \
    curl -Lf -o /tmp/openssh-clients.rpm http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/openssh-clients-8.0p1-5.el8.x86_64.rpm; \
    curl -Lf -o /tmp/perl-libxml-perl.rpm http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/perl-libxml-perl-0.08-33.el8.noarch.rpm; \
    curl -Lf -o /tmp/perl-DBD-Pg.rpm http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/perl-DBD-Pg-3.7.4-4.module_el8.3.0+426+0b4e9c0a.x86_64.rpm; \
    curl -Lf -o /tmp/perl-XML-Parser.rpm http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/perl-XML-Parser-2.44-11.el8.x86_64.rpm; \
    curl -Lf -o /tmp/perl-DBI.rpm http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/perl-DBI-1.641-3.module_el8.1.0+199+8f0a6bbd.x86_64.rpm; \
    curl -Lf -o /tmp/llvm-libs.rpm http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/llvm-libs-10.0.1-3.module_el8.3.0+467+cb298d5b.x86_64.rpm; \
    curl -Lf -o /tmp/llvm.rpm http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/llvm-10.0.1-3.module_el8.3.0+467+cb298d5b.x86_64.rpm; \
    rpmkeys --checksig /tmp/perl-DBI.rpm /tmp/perl-DBI.rpm /tmp/perl-XML-Parser.rpm /tmp/perl-libxml-perl.rpm /tmp/perl-DBD-Pg.rpm /tmp/openssh-server.rpm /tmp/openssh-clients.rpm; \
    rpm -i /tmp/perl-DBI.rpm /tmp/llvm-libs.rpm /tmp/llvm.rpm /tmp/perl-DBI.rpm /tmp/perl-XML-Parser.rpm /tmp/perl-libxml-perl.rpm /tmp/perl-DBD-Pg.rpm /tmp/openssh-server.rpm /tmp/openssh-clients.rpm; \
    rm -rf /tmp/perl-DBI.rpm /tmp/llvm-libs.rpm /tmp/llvm.rpm /tmp/perl-DBI.rpm /tmp/perl-XML-Parser.rpm /tmp/perl-libxml-perl.rpm /tmp/perl-DBD-Pg.rpm /tmp/openssh-server.rpm /tmp/openssh-clients.rpm

RUN  set -ex; \
     microdnf -y install \
		--enablerepo="epel" \
		percona-pgaudit \
		percona-pgaudit${PG_MAJOR//.}_set_user \
		percona-pgbackrest \
		percona-postgresql${PG_MAJOR//.}-contrib \
		percona-postgresql${PG_MAJOR//.}-server \
		percona-postgresql${PG_MAJOR//.}-plpython* \
		percona-pg-stat-monitor${PG_MAJOR//.} \
		percona-postgresql${PG_MAJOR//.}-llvmjit \
		psmisc \
		rsync \
		perl \
		percona-wal2json${PG_MAJOR//.} \
		file \
		unzip; \
#	 microdnf -y install \
#		--setopt=tsflags='' \
#		--enablerepo="epel" \
#		pgaudit_analyze; \
	microdnf -y clean all --enablerepo="epel"

# ===== Steps unique to this image after here =====
LABEL name="postgres" \
	summary="PostgreSQL ${PG_MAJOR}" \
	description="Allows multiple deployment methods for PostgreSQL, including basic single primary, streaming replication with synchronous and asynchronous replicas. Includes utilities and tools for Auditing (percona-pgaudit), statement tracking,  Backup/Restore (percona-pgbackrest)." \
	io.k8s.description="Percona PostgreSQL is the trusted open source distribution of PostgreSQL" \
	io.k8s.display-name="Percona PostgreSQL" \
	io.openshift.tags="postgresql,postgres,sql,nosql,database,percona"

# set up crunchy directory
RUN mkdir -p /opt/crunchy/bin /opt/crunchy/conf /pgdata /pgwal /pgconf /recover /backrestrepo

RUN chown -R postgres:postgres /opt/crunchy /var/lib/pgsql \
		/pgdata /pgwal /pgconf /recover /backrestrepo &&  \
	chmod -R g=u /opt/crunchy /var/lib/pgsql \
		/pgdata /pgwal /pgconf /recover /backrestrepo

# open up the postgres port
EXPOSE 5432

ADD bin/postgres_common /opt/crunchy/bin
ADD bin/common /opt/crunchy/bin
ADD conf/postgres_common /opt/crunchy/conf
#ADD tools/pgmonitor/exporter/postgres /opt/crunchy/bin/modules/pgexporter

RUN chmod g=u /etc/passwd && \
	chmod g=u /etc/group

RUN mkdir /.ssh && chown 26:0 /.ssh && chmod g+rwx /.ssh && rm -f /run/nologin

# add volumes to allow override of pg_hba.conf and postgresql.conf
# add volumes to offer a restore feature
# add volumes to allow storage of postgres WAL segment files
# add volumes to locate WAL files to recover with
# add volumes for pgbackrest to write to
# The VOLUME directive must appear after all RUN directives to ensure the proper
# volume permissions are applied when building the image
VOLUME ["/sshd", "/pgconf", "/pgdata", "/pgwal", "/recover", "/backrestrepo"]

ENTRYPOINT ["/opt/crunchy/bin/uid_postgres.sh"]

USER 26

CMD ["/opt/crunchy/bin/start.sh"]
