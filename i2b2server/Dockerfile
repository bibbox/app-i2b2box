#This Dockerfile creates an I2B2 instance with all the cells on a single machine
#To build: docker build  --tag=newi2b2-server --rm=true .
#To run: docker run --name=foorun -p 8080:8080 -p 9090:9090 -p 9990:9990 -p 9009:9009  i2b2-server
#To verify installation: load http://localhost:8080/i2b2/services/listServices

#ARG JBOSS_IMAGE=crs4/jboss_wildfly:10.0.0.Final

FROM crs4/jboss_wildfly:10.0.0.Final
## following ARGs need to be placed after FROM
ARG DBNODE
ARG I2B2NODE
ARG DB_CONNECTION_URL=${DBNODE}:5432/i2b2
ARG DB_PM_USER=i2b2pm
ARG DB_PM_PASSWORD=demouser
ARG DB_ONT_USER=i2b2metadata
ARG DB_ONT_PASSWORD=demouser
ARG DB_CRC_USER=i2b2demodata
ARG DB_CRC_PASSWORD=demouser
ARG DB_HIVE_USER=i2b2hive
ARG DB_HIVE_PASSWORD=demouser
ARG DB_WORK_USER=i2b2workdata
ARG DB_WORK_PASSWORD=demouser
ARG ADMIN_DOMAIN=i2b2demo
ARG ADMIN_NAME=i2b2demo
ARG ADMIN_URL_CELL_PM=http://${I2B2NODE}:9090/i2b2/services/PMService/
ARG JBOSS_CONSOLE_LEVEL=INFO

ARG I2B2_HTTP_PORT=9090
ARG I2B2_AJP_PORT=9009
ARG ADMIN_PORT=8080
ARG I2B2_SERVERNAME=${I2B2NODE}

ENV I2B2_HOME /opt/i2b2
ENV JBOSS_HTTP_PORT ${I2B2_HTTP_PORT}
ENV JBOSS_AJP_PORT ${I2B2_AJP_PORT}


USER root

RUN apk update && apk add wget && apk add  ca-certificates && wget https://space.crs4.it/s/G9PptQK0Fdwh54G/download -O /i2b2core-src-1708.zip 

#COPY resources/i2b2core-src-1708.zip /
COPY resources/start_server.sh /
COPY resources/ont-ds.xml /
COPY resources/pm-ds.xml /
COPY resources/crc-ds.xml /
COPY resources/work-ds.xml /
COPY resources/i2b2_config_data.js /
COPY resources/i2b2core-src-1708.zip.md5 /

RUN cd / && md5sum -c -s i2b2core-src-1708.zip.md5


RUN mkdir -p ${I2B2_HOME} && \
    unzip /i2b2core-src-1708.zip -d ${I2B2_HOME} && \
    rm /i2b2core-src-1708.zip && \
    chmod +x /start_server.sh


############## CONFIGURATION AND DEPLOY of SERVER-COMMON ###############

WORKDIR ${I2B2_HOME}/edu.harvard.i2b2.server-common
RUN sed  -i  "s/jboss.home/#jboss.home/" build.properties && \
    sed  -i  "/#jboss.home/ a jboss.home=${WILDFLY_HOME}/" build.properties && \
    ant clean dist deploy jboss_pre_deployment_setup


##############  Project Management cell ###############

WORKDIR ${I2B2_HOME}/edu.harvard.i2b2.pm

RUN sed  -i  "s/jboss.home/#jboss.home/" build.properties && \
    sed  -i  "/#jboss.home/ a jboss.home=${WILDFLY_HOME}" build.properties && \
    sed  -e "s!DB_CONNECTION_URL!${DB_CONNECTION_URL}!" \
         -e "s!DB_PM_USER!${DB_PM_USER}!" \
         -e "s!DB_PM_PASSWORD!${DB_PM_PASSWORD}!" \
         < /pm-ds.xml > etc/jboss/pm-ds.xml && \
    ant -f master_build.xml clean build-all deploy


##############  Admin setup  ###############

ENV ADMIN_HOME /var/www/localhost/htdocs/
COPY resources/index.php /

RUN apk update && apk add apache2 php5-apache2 curl php5-curl && \
    mkdir /run/apache2/ && \
    cp -r  ${I2B2_HOME}/admin /var/www/localhost/htdocs/ && \
    sed -e "s!ADMIN_DOMAIN!${ADMIN_DOMAIN}!" \
        -e "s!ADMIN_NAME!${ADMIN_NAME}!" \
        -e "s!ADMIN_URL_CELL_PM!${ADMIN_URL_CELL_PM}!" \
        < /i2b2_config_data.js > ${ADMIN_HOME}/admin/i2b2_config_data.js && \
    sed -i "s/Listen 80/Listen ${ADMIN_PORT}/" /etc/apache2/httpd.conf && \
    mv /index.php ${ADMIN_HOME}/admin/ && \
    sed -i "s/I2B2_SERVERNAME/${I2B2_SERVERNAME}/" ${ADMIN_HOME}/admin/index.php && \
    sed -i "s/I2B2_HTTP_PORT/${I2B2_HTTP_PORT}/" ${ADMIN_HOME}/admin/index.php


##############  Ontology setup  ###############

WORKDIR ${I2B2_HOME}/edu.harvard.i2b2.ontology

RUN sed -i "s/wildfly-10.0.0.Final/wildfly/" build.properties && \
    sed -i "s/wildfly-10.0.0.Final/wildfly/" etc/spring/ontology_application_directory.properties && \
    sed  -e "s!DB_CONNECTION_URL!${DB_CONNECTION_URL}!" \
         -e "s!DB_ONT_USER!${DB_ONT_USER}!" \
         -e "s!DB_ONT_PASSWORD!${DB_ONT_PASSWORD}!" \
         -e "s!DB_HIVE_USER!${DB_HIVE_USER}!" \
         -e "s!DB_HIVE_PASSWORD!${DB_HIVE_PASSWORD}!" \
         < /ont-ds.xml > etc/jboss/ont-ds.xml && \
    ant -f master_build.xml clean build-all deploy


##############  CRC setup  ###############

WORKDIR ${I2B2_HOME}/edu.harvard.i2b2.crc

RUN sed -i -e "s/ORACLE/PostgreSQL/" etc/spring/crc.properties && \
    sed -i -e "s/ORACLE/PostgreSQL/" etc/spring/edu.harvard.i2b2.crc.loader.properties && \
    sed -i "s/wildfly-10.0.0.Final/wildfly/" build.properties && \
    sed -i "s/wildfly-10.0.0.Final/wildfly/" etc/spring/crc_application_directory.properties && \
    sed  -e "s!DB_CONNECTION_URL!${DB_CONNECTION_URL}!" \
         -e "s!DB_CRC_USER!${DB_CRC_USER}!" \
         -e "s!DB_CRC_PASSWORD!${DB_CRC_PASSWORD}!" \
         -e "s!DB_HIVE_USER!${DB_HIVE_USER}!" \
         -e "s!DB_HIVE_PASSWORD!${DB_HIVE_PASSWORD}!" \
         < /crc-ds.xml > etc/jboss/crc-ds.xml && \
    ant -f master_build.xml clean build-all deploy


##############  WORKPLACE setup  ###############

WORKDIR ${I2B2_HOME}/edu.harvard.i2b2.workplace

RUN sed -i -e "s/ORACLE/PostgreSQL/" etc/spring/workplace.properties && \
    sed -i -e "s/ORACLE/PostgreSQL/" etc/spring/workplace_application_directory.properties && \
    sed -i "s/wildfly-10.0.0.Final/wildfly/" build.properties && \
    sed -i "s/wildfly-10.0.0.Final/wildfly/" etc/spring/workplace_application_directory.properties && \
    sed  -e "s!DB_CONNECTION_URL!${DB_CONNECTION_URL}!" \
         -e "s!DB_WORK_USER!${DB_WORK_USER}!" \
         -e "s!DB_WORK_PASSWORD!${DB_WORK_PASSWORD}!" \
         -e "s!DB_HIVE_USER!${DB_HIVE_USER}!" \
         -e "s!DB_HIVE_PASSWORD!${DB_HIVE_PASSWORD}!" \
         < /work-ds.xml > etc/jboss/work-ds.xml && \
    ant -f master_build.xml clean build-all deploy


##############  FR setup  ###############

WORKDIR ${I2B2_HOME}/edu.harvard.i2b2.fr

RUN sed -i "s/wildfly-10.0.0.Final/wildfly/" build.properties && \
    sed -i "s/wildfly-10.0.0.Final/wildfly/" etc/spring/fr_application_directory.properties && \
    ant -f master_build.xml clean build-all deploy


EXPOSE ${ADMIN_PORT} 9090 9990 9009

USER root
CMD ["/start_server.sh"]

