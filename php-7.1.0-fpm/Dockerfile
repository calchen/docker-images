FROM centos:7
MAINTAINER "陈恺垣 contact@chenky.com"

ENV php_hash 9e84c5b13005c56374730edf534fe216f6a2e63792a9703d4b894e770bbccbae

# 安装依赖
RUN yum update -y && \
    yum install -y wget make gcc && \
    # php 依赖
    yum install -y libxml2-devel openssl-devel curl-devel && \
    yum -y clean all

# 下载，编译，安装
RUN wget http://cn2.php.net/distributions/php-7.1.0.tar.gz && \
    echo "$php_hash php-7.1.0.tar.gz" | sha256sum -c - && \
    tar -xf php-7.1.0.tar.gz && \
    # php7 安转
    cd php-7.1.0 && \
    ./configure \
        --enable-fpm \
        --with-openssl \
        --enable-zip \
        --enable-mbstring \
        --enable-opcache \
        --with-pdo-mysql \
        --with-curl \
        --with-config-file-scan-dir=/usr/local/lib/conf.d && \
    make && \
    make install && \
    /usr/bin/cp -f sapi/fpm/php-fpm /usr/local/bin/php-fpm && \
    mkdir -p /usr/local/lib/conf.d && \
    cd ../ && \
    # 清理
    rm -rf php-7.1.0 php-7.1.0.tar.gz

# 增加用户 apache
RUN groupadd apache && \
    mkdir -p /var/www/html && \
    useradd  -g apache -s /bin/bash -d /var/www/html apache

# php.ini
COPY ./config/php.ini /usr/local/lib/php.ini

# php-fpm
COPY ./config/php-fpm.conf /usr/local/etc/php-fpm.conf

# php-fpm pool
COPY ./config/php-fpm-www.conf /usr/local/etc/php-fpm.d/www.conf

RUN mkdir -p /usr/local/var/www/log && \
    touch /usr/local/var/www/log/www.access.log

# 入口
COPY ./docker-entrypoint /usr/local/bin/docker-entrypoint
RUN chmod +x /usr/local/bin/docker-entrypoint
ENTRYPOINT ["docker-entrypoint"]

CMD ["php-fpm"]