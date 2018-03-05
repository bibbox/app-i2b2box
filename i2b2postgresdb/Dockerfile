# based on https://github.com/yobasystems/alpine-postgres/blob/master/Dockerfile
# see https://github.com/yobasystems/alpine-postgres for details on configuration
# Example: 
#  docker run --name some-postgres -e POSTGRES_PASSWORD=foo -p 5432:5432 <i2b2-db>

# POSTGRES_PASSWORD: This environment variable is recommended for you to use the
# PostgreSQL image. This environment variable sets the superuser password for
# PostgreSQL. The default superuser is defined by the POSTGRES_USER environment
# variable. In the above example, it is being set to "foo".

# POSTGRES_USER: This optional environment variable is used in conjunction with
# POSTGRES_PASSWORD to set a user and its password. This variable will create the
# specified user with superuser power and a database with the same name. If it is
# not specified, then the default user of "postgres" will be used.

# PGDATA: This optional environment variable can be used to define another
# location - like a subdirectory - for the database files. The default is
# /var/lib/postgresql/data, but if the data volume you're using is a fs mountpoint
# (like with GCE persistent disks), Postgres initdb recommends a subdirectory (for
# example /var/lib/postgresql/data/pgdata ) be created to contain the data.

# POSTGRES_DB: This optional environment variable can be used to define a
# different name for the default database that is created when the image is first
# started. If it is not specified, then the value of POSTGRES_USER will be used.

# POSTGRES_INITDB_ARGS: This optional environment variable can be used to send
# arguments to postgres initdb. The value is a space separated string of arguments
# as postgres initdb would expect them. This is useful for adding functionality
# like data page checksums: -e POSTGRES_INITDB_ARGS="--data-checksums".

# To change the configuration file in a container do
#  docker exec -it <db-container> -u ${POSTGRES_USER} /bin/sh
#  container_shell$ vi ${PGDATA}/postgresql.conf
#  container_shell$ pg_ctl reload
#  container_shell$ cntl-D


FROM alpine:3.5
MAINTAINER Alfred E. Neumann

ENV LANG en_US.utf8
ENV PGDATA /var/lib/postgresql/data

#ENV  POSTGRES_USER postgres
#ENV  POSTGRES_PASSWORD postgres
#ENV  POSTGRES_DB i2b2

COPY resources/docker-entrypoint.sh /

RUN apk update && \
    apk add su-exec tzdata libpq postgresql-client \
            postgresql postgresql-contrib && \
    mkdir /docker-entrypoint-initdb.d && \
    rm -rf /var/cache/apk/* && \
    chmod +x /docker-entrypoint.sh

#VOLUME /var/lib/postgresql/data



RUN mkdir /myresources && apk update && apk add python python-dev py-psycopg2 py-pip build-base && pip install pyyaml && rm -rf /var/cache/apk/*

COPY ./resources/data /myresources/

ARG I2B2NODE

RUN sed  -i  "s/i2b2-server/${I2B2NODE}/" /myresources/db_data_99_PMD_redefine_hosts.yml

ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 5432
CMD ["postgres"]
