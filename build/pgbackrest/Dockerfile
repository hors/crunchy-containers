FROM golang:1.13 AS go_builder
WORKDIR /go/src/github.com/hors/crunchy-containers/src

COPY go.* /go/src/github.com/hors/crunchy-containers/src/
COPY cmd/pgbackrest/main.go /go/src/github.com/hors/crunchy-containers/src/
RUN go build -o pgbackrest

FROM perconalab/percona-postgres-operator:main-base

RUN set -ex; \
    microdnf -y install  \
	    openssh \
        systemd \
        libpq \
        libedit 

RUN set -ex; \
    curl -Lf -o /tmp/perl-DBI.rpm http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/perl-DBI-1.641-3.module_el8.1.0+199+8f0a6bbd.x86_64.rpm; \
    curl -Lf -o /tmp/perl-XML-Parser.rpm http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/perl-XML-Parser-2.44-11.el8.x86_64.rpm; \
    curl -Lf -o /tmp/openssh-server.rpm http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/openssh-server-8.0p1-5.el8.x86_64.rpm; \
    curl -Lf -o /tmp/openssh-clients.rpm http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/openssh-clients-8.0p1-5.el8.x86_64.rpm; \
    curl -Lf -o /tmp/perl-libxml-perl.rpm http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/perl-libxml-perl-0.08-33.el8.noarch.rpm; \
    curl -Lf -o /tmp/perl-DBD-Pg.rpm http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/perl-DBD-Pg-3.7.4-4.module_el8.3.0+426+0b4e9c0a.x86_64.rpm; \
    rpmkeys --checksig /tmp/perl-DBI.rpm /tmp/perl-XML-Parser.rpm /tmp/perl-libxml-perl.rpm /tmp/perl-DBD-Pg.rpm /tmp/openssh-server.rpm /tmp/openssh-clients.rpm; \
    rpm -i /tmp/perl-DBI.rpm /tmp/perl-XML-Parser.rpm /tmp/openssh-server.rpm /tmp/openssh-clients.rpm /tmp/perl-libxml-perl.rpm /tmp/perl-DBD-Pg.rpm; \
    rm -rf /tmp/perl-DBI.rpm /tmp/perl-XML-Parser.rpm /tmp/openssh-server.rpm /tmp/openssh-clients.rpm /tmp/perl-libxml-perl.rpm /tmp/perl-DBD-Pg.rpm

# Run postgres install in separate transaction ahead of backrest for postgres user
RUN set -ex; \
    microdnf -y install  \
	   percona-pgbackrest; \
	microdnf -y clean all


# add postgres user and group
RUN groupadd postgres -g 26 && useradd postgres -u 26 -g 26

# set up crunchy directory
RUN mkdir -p /opt/crunchy/bin /opt/crunchy/conf /pgdata /backrestrepo \
	/var/log/pgbackrest

RUN pwd; ls -al
# add pgbackrest-restore files
COPY bin/pgbackrest-restore /opt/crunchy/bin
COPY conf/pgbackrest-restore /opt/crunchy/conf

# add pgbackrest files
COPY --from=go_builder /go/src/github.com/hors/crunchy-containers/src/pgbackrest /opt/crunchy/bin/
# add pgbackrest-common files
COPY bin/common /opt/crunchy/bin
COPY bin/pgbackrest-common /opt/crunchy/bin

# set user and group ownership
RUN chown -R postgres:postgres /opt/crunchy  \
	/backrestrepo /var/log/pgbackrest /pgdata

# add the pgbackrest-repo specific files and directories, and
# set the appropriate permissions and ownership
COPY bin/pgbackrest-repo /usr/local/bin
RUN chmod +x /usr/local/bin/pgbackrest-repo.sh /usr/local/bin/archive-push-s3.sh; \
	mkdir -p /etc/pgbackrest; \
	chown -R postgres:postgres /etc/pgbackrest

RUN chmod g=u /etc/passwd; \
	chmod g=u /etc/group; \
	chmod -R g=u /etc/pgbackrest; \
	rm -f /run/nologin

RUN mkdir /.ssh && chown postgres:postgres /.ssh && chmod o+rwx /.ssh

# create necessary volumes for all modes
#
# /sshd and /backrestrepo required for the pgBackRest repo mode
# /pgdata required for the pgbackrest-restore mode
#
# The VOLUME directive must appear after all RUN directives to ensure the proper
# volume permissions are applied when building the image
VOLUME ["/sshd", "/pgdata", "/backrestrepo"]

USER 26

ENTRYPOINT ["/opt/crunchy/bin/uid_postgres.sh"]

CMD ["/opt/crunchy/bin/start.sh"]
