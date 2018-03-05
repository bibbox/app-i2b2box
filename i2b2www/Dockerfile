FROM alpine:3.5

# This create a i2b2-www image as described in section 7 of the installation
# instructions.

ARG I2B2_VERSION=1.7.8
ARG I2B2_VERSION_SHORT=1708
ARG I2B2_DOMAIN=i2b2demo
ARG I2B2_NAME=i2b2demo
ARG I2B2_ALLOW_ANALYSIS=true
ARG I2B2_DEBUG=true
ARG I2B2NODE
ARG I2B2_URL_CELL_PM=http://${I2B2NODE}:9090/i2b2/services/PMService/

ENV I2B2_HOME /opt/i2b2
ENV I2B2_WEBCLIENT i2b2webclient-${I2B2_VERSION_SHORT}.zip

RUN apk update && apk add wget && apk add  ca-certificates && wget https://space.crs4.it/s/ewH0Zr5jB5LdNNm/download -O /${I2B2_WEBCLIENT}

#COPY resources/${I2B2_WEBCLIENT} /
COPY resources/${I2B2_WEBCLIENT}.md5 /

RUN cd / && md5sum -c -s ${I2B2_WEBCLIENT}.md5


RUN mkdir -p ${I2B2_HOME} && \
    unzip /${I2B2_WEBCLIENT} -d ${I2B2_HOME} && \
    rm /${I2B2_WEBCLIENT}

ENV WEB_HOME /var/www/localhost/htdocs/
RUN apk update && apk add apache2 php5-apache2 curl php5-curl && \
    mkdir /run/apache2/ && \
    cp -r  ${I2B2_HOME}/i2b2webclient* ${WEB_HOME}/webclient && \
    echo "{ \
        urlProxy: \"index.php\", \
        urlFramework: \"js-i2b2/\", \
        lstDomains: [ \
            { domain: \"${I2B2_DOMAIN}\", \
              name: \"${I2B2_NAME}\", \
              urlCellPM: \"${I2B2_URL_CELL_PM}\", \
              allowAnalysis: ${I2B2_ALLOW_ANALYSIS}, \
              debug: ${I2B2_DEBUG} \
            } \
        ] \
    }" > ${WEB_HOME}/webclient/i2b2_config_data.js

RUN sed -i "s/127.0.0.1/${I2B2NODE}/" ${WEB_HOME}/webclient/index.php
RUN sed -i "s|/rest/|/services/|" ${WEB_HOME}/webclient/index.php


EXPOSE 80
ENTRYPOINT ["/usr/sbin/httpd"]
CMD ["-k",  "start", "-D", "FOREGROUND"]
