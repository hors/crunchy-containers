FROM perconalab/percona-postgres-operator:main-pgbackrest

# run the following at root user
USER 0

# add pgbackrest user and group
RUN groupadd pgbackrest -g 2000 && useradd pgbackrest -u 2000 -g 2000

# set ownership and permissions for pgbackrest user and group
RUN chown -R pgbackrest:pgbackrest /opt/crunchy; \
	chown -R pgbackrest /etc/pgbackrest

# add pgbackrest user and group script
COPY bin/pgbackrest-repo/uid_pgbackrest.sh /opt/crunchy/bin

# set permissions for pgbackrest user
RUN chmod g=u /etc/passwd; \
	chmod g=u /etc/group; \
	chmod -R g=u /etc/pgbackrest

# set ownership to pgbackrest user
RUN chown pgbackrest:pgbackrest /.ssh && chmod o+rwx /.ssh

# set user to pgbackrest user
USER 2000

ENTRYPOINT ["/opt/crunchy/bin/uid_pgbackrest.sh"]

CMD ["pgbackrest-repo.sh"]
