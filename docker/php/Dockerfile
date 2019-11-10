FROM php:7.2-fpm-alpine

ENV LD_LIBRARY_PATH /opt/oci8/instantclient_12_2
ENV ORACLE_HOME /opt/oci8/instantclient_12_2

# Set working directory
WORKDIR /var/www

RUN docker-php-ext-install pdo pdo_mysql && \
    apk add bash git autoconf gcc make g++ zlib-dev libaio libnsl musl-dev libc6-compat gcompat libtool

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Install oracleclient
RUN ln -s /usr/lib/libnsl.so.2 /usr/lib/libnsl.so.1

RUN mkdir -p /opt/oci8

COPY ./oci8/instantclient-basic-linux.x64-12.2.0.1.0.zip /opt/oci8
COPY ./oci8/instantclient-sdk-linux.x64-12.2.0.1.0.zip /opt/oci8
COPY ./oci8/instantclient-sqlplus-linux.x64-12.2.0.1.0.zip /opt/oci8

RUN cd /opt/oci8 \
    && unzip instantclient-sdk-linux.x64-12.2.0.1.0.zip \
    && unzip instantclient-basic-linux.x64-12.2.0.1.0.zip \
    && unzip instantclient-sqlplus-linux.x64-12.2.0.1.0.zip \
    && cd instantclient_12_2/ \
    && ln -s libclntsh.so.12.1 libclntsh.so \
    && ln -s libocci.so.12.1 libocci.so

# Install Oracle extensions
RUN ln -s /lib/libc.so.6 /usr/lib/libresolv.so.2 \
    && ln -s /lib/ld-linux-x86-64.so.2 ${ORACLE_HOME}/ld-linux-x86-64.so.2 \
    && ln -s ${ORACLE_HOME}/lib* /usr/lib \
    && docker-php-ext-configure pdo_oci --with-pdo-oci=instantclient,/opt/oci8/instantclient_12_2,12.2 \
    && echo 'instantclient,/opt/oci8/instantclient_12_2/' | pecl install oci8 \
    && docker-php-ext-install pdo_oci \
    && docker-php-ext-enable oci8

# Add user for laravel application
RUN addgroup -g 1000 www
RUN adduser -S -G www -u 1000 www

# Copy helpers binaries
RUN mkdir /tmp/bin
COPY /bin/* /tmp/bin/
RUN chmod +x /tmp/bin/*
RUN mv /tmp/bin/* /bin/

# Change current user to www
USER www

# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["startup"]