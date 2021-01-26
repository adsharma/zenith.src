FROM archlinux:base-devel
MAINTAINER Arun Sharma <arun@sharma-home.net>

RUN pacman -Sy wget postgresql --noconfirm && (yes | pacman -Scc || :)

VOLUME /var/lib/postgresql/data

# grab gosu for easy step-down from root
# https://github.com/tianon/gosu/releases
ENV GOSU_VERSION 1.12
RUN set -eux; \
	dpkgArch="amd64"; \
	wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch"; \
	wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch.asc"; \
	chmod +x /usr/local/bin/gosu; \
	gosu --version; \
	gosu nobody true

RUN mkdir /docker-entrypoint-initdb.d

ENV PG_MAJOR 13
ENV PGDATA /var/lib/postgresql/data

# setup postgresql
RUN mkdir -p /var/run/postgresql && chown -R postgres:postgres /var/run/postgresql && chmod 2777 /var/run/postgresql

COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]

# We set the default STOPSIGNAL to SIGINT, which corresponds to what PostgreSQL
# calls "Fast Shutdown mode" wherein new connections are disallowed and any
# in-progress transactions are aborted, allowing PostgreSQL to stop cleanly and
# flush tables to disk, which is the best compromise available to avoid data
# corruption.
#
# Users who know their applications do not keep open long-lived idle connections
# may way to use a value of SIGTERM instead, which corresponds to "Smart
# Shutdown mode" in which any existing sessions are allowed to finish and the
# server stops when all sessions are terminated.
#
# See https://www.postgresql.org/docs/12/server-shutdown.html for more details
# about available PostgreSQL server shutdown signals.
#
# See also https://www.postgresql.org/docs/12/server-start.html for further
# justification of this as the default value, namely that the example (and
# shipped) systemd service files use the "Fast Shutdown mode" for service
# termination.
#
STOPSIGNAL SIGINT
#
# An additional setting that is recommended for all users regardless of this
# value is the runtime "--stop-timeout" (or your orchestrator/runtime's
# equivalent) for controlling how long to wait between sending the defined
# STOPSIGNAL and sending SIGKILL (which is likely to cause data corruption).
#
# The default in most runtimes (such as Docker) is 10 seconds, and the
# documentation at https://www.postgresql.org/docs/12/server-start.html notes
# that even 90 seconds may not be long enough in many instances.
EXPOSE 5432
CMD ["postgres"]
