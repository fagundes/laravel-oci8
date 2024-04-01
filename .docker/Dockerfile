FROM ubuntu:22.04

ARG WWWGROUP

WORKDIR /var/www/html

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=UTC
ENV SUPERVISOR_PHP_COMMAND="/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan serve --host=0.0.0.0 --port=80"
ENV SUPERVISOR_PHP_USER="sail"

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# install deps
RUN apt-get update \
    && mkdir -p /etc/apt/keyrings \
    && apt-get install -y gnupg gosu curl ca-certificates zip unzip git supervisor build-essential libcap2-bin wget libaio1  \
    && apt-get install -y  \
    && curl -sS 'https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x14aa40ec0831756756d7f66c4f4ea0aae5267a6c' | gpg --dearmor | tee /etc/apt/keyrings/ppa_ondrej_php.gpg > /dev/null \
    && echo "deb [signed-by=/etc/apt/keyrings/ppa_ondrej_php.gpg] https://ppa.launchpadcontent.net/ondrej/php/ubuntu jammy main" > /etc/apt/sources.list.d/ppa_ondrej_php.list \
    && apt-get update \
    && apt-get install -y php8.2-cli php8.2-dev \
       php8.2-dom php8.2-curl \
       php8.2-xml php8.2-mbstring \
       php8.2-zip php8.2-pdo \
       php8.2-memcached php8.2-gd  \
    && curl -sLS https://getcomposer.org/installer | php -- --install-dir=/usr/bin/ --filename=composer \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN setcap "cap_net_bind_service=+ep" /usr/bin/php8.2

RUN groupadd --force -g $WWWGROUP sail
RUN useradd -ms /bin/bash --no-user-group -g $WWWGROUP -u 1337 sail

# install oci8 libs & extension
RUN mkdir -p /opt/oracle

RUN wget https://github.com/bumpx/oracle-instantclient/raw/master/instantclient-basic-linux.x64-12.1.0.2.0.zip
RUN wget https://github.com/bumpx/oracle-instantclient/raw/master/instantclient-sdk-linux.x64-12.1.0.2.0.zip

RUN unzip -o ./instantclient-basic-linux.x64-12.1.0.2.0.zip -d /opt/oracle
RUN unzip -o ./instantclient-sdk-linux.x64-12.1.0.2.0.zip -d /opt/oracle

RUN ln -s /opt/oracle/instantclient/sqlplus /usr/bin/sqlplus
RUN ln -s /opt/oracle/instantclient_12_1 /opt/oracle/instantclient
RUN ln -s /opt/oracle/instantclient/libclntsh.so.12.1 /opt/oracle/instantclient/libclntsh.so
RUN ln -s /opt/oracle/instantclient/libocci.so.12.1 /opt/oracle/instantclient/libocci.so

RUN sh -c "echo 'instantclient,/opt/oracle/instantclient' | pecl install oci8"

# setup ld library path
RUN sh -c "echo '/opt/oracle/instantclient' >> /etc/ld.so.conf"
RUN ldconfig

# enable oci8 extension
RUN sh -c "echo 'extension=oci8.so' >> /etc/php/8.2/cli/php.ini"

COPY .docker/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY .docker/start-container /usr/local/bin/start-container
RUN chmod +x /usr/local/bin/start-container

ENTRYPOINT ["start-container"]