FROM centos:7
MAINTAINER "陈恺垣 contact@chenky.com"

ENV openssl_hash e7aff292be21c259c6af26469c7a9b3ba26e9abaaffd325e3dccc9785256c431
ENV headers_more_nginx_module_hash c6d9dab8ea1fc997031007e2e8f47cced01417e203cd88d53a9fe9f6ae138720
ENV nginx_hash 1045ac4987a396e2fa5d0011daf8987b612dd2f05181b67507da68cbe7d765c2

# 安装依赖
RUN yum update -y && \
    yum install -y wget make gcc && \
    # openssl 依赖
    yum install -y perl && \
    # nginx 依赖
    yum install -y pcre-devel zlib-devel && \
    yum -y clean all

# openssl 下载，编译，安装
RUN wget https://www.openssl.org/source/openssl-1.0.2j.tar.gz && \
    echo "$openssl_hash openssl-1.0.2j.tar.gz" | sha256sum -c - && \
    tar -xf openssl-1.0.2j.tar.gz && \
    cd openssl-1.0.2j && \
    ./config && \
    make && \
    make install && \
    ln -sf /usr/local/ssl/bin/openssl /usr/bin/openssl

    # nginx 模块 headers-more-nginx-module
    # https://github.com/openresty/headers-more-nginx-module
RUN wget https://github.com/openresty/headers-more-nginx-module/archive/v0.32.tar.gz -O headers-more-nginx-module-0.32.tar.gz && \
    echo "$headers_more_nginx_module_hash headers-more-nginx-module-0.32.tar.gz" | sha256sum -c - && \
    tar -xf headers-more-nginx-module-0.32.tar.gz && \
    # nginx 下载，编译，安装
    wget http://nginx.org/download/nginx-1.10.2.tar.gz && \
    echo "$nginx_hash nginx-1.10.2.tar.gz" | sha256sum -c - && \
    tar -xf nginx-1.10.2.tar.gz && \
    cd nginx-1.10.2 && \
    ./configure \
        --prefix=/etc/nginx \
        --sbin-path=/usr/sbin/nginx \
        --modules-path=/usr/lib64/nginx/modules \
        --conf-path=/etc/nginx/nginx.conf \
        --error-log-path=/var/log/nginx/error.log \
        --http-log-path=/var/log/nginx/access.log \
        --pid-path=/var/run/nginx.pid \
        --lock-path=/var/run/nginx.lock \
        --http-client-body-temp-path=/var/cache/nginx/client_temp \
        --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
        --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
        --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
        --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
        --user=nginx \
        --group=nginx \
        --with-file-aio \
        --with-threads \
        --with-ipv6 \
        --with-http_addition_module \
        --with-http_auth_request_module \
        --with-http_dav_module \
        --with-http_flv_module \
        --with-http_gunzip_module \
        --with-http_gzip_static_module \
        --with-http_mp4_module \
        --with-http_random_index_module \
        --with-http_realip_module \
        --with-http_secure_link_module \
        --with-http_slice_module \
        --with-http_ssl_module \
        --with-http_stub_status_module \
        --with-http_sub_module \
        --with-http_v2_module \
        --with-mail \
        --with-mail_ssl_module \
        --with-stream \
        --with-stream_ssl_module \
        --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic' \
        --with-openssl=/openssl-1.0.2j \
        --add-module=/headers-more-nginx-module-0.32 && \
    make && \
    make install && \
    cd ../ && \
    # 清理
    rm -rf openssl-1.0.2j openssl-1.0.2j.tar.gz && \
    rm -rf headers-more-nginx-module-0.32 headers-more-nginx-module-0.32.tar.gz && \
    rm -rf nginx-1.10.2 nginx-1.10.2.tar.gz

# 增加用户 apache
RUN groupadd nginx && \
    mkdir -p /var/www/html && \
    useradd  -g nginx -s /bin/bash -d /var/www/html nginx

# nginx config
COPY ./config/nginx.conf ./config/proxy.conf /etc/nginx/
COPY ./config/default-www.conf /etc/nginx/conf.d/default.conf
RUN mkdir -p /etc/nginx/conf.d && \
    mkdir -p /var/cache/nginx/ && \
    mkdir -p /usr/local/var/www/log && \
    touch /usr/local/var/www/log/www.access.log

# 入口
COPY ./docker-entrypoint /usr/local/bin/docker-entrypoint
RUN chmod +x /usr/local/bin/docker-entrypoint
ENTRYPOINT ["docker-entrypoint"]

CMD ["nginx"]