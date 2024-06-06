FROM php:8.2-fpm-alpine
COPY php.ini /usr/local/etc/php/php.ini
#COPY .ssh /root/.ssh

LABEL maintainer="xujunfeng"
WORKDIR /www

ENV GOPATH /go
ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH
ENV GOPROXY https://goproxy.io,direct

#安装composer
RUN php -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');" \
     && php composer-setup.php \
     && php -r "unlink('composer-setup.php');" \
     && mv composer.phar /usr/local/bin/composer \
     && composer config -g repo.packagist composer https://mirrors.aliyun.com/composer


# 更新包管理工具的包列表，安装一些基本工具
RUN sed -i "s/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g" /etc/apk/repositories  \
    && set -ex \
    && apk update \
    && apk upgrade  \
    && apk add --no-cache  build-base perl autoconf g++ libtool make curl-dev go  \
    && apk add --no-cache  libcurl libzip-dev git icu-dev rabbitmq-c-dev protobuf openssl-dev openssl libffi-dev postgresql-dev c-ares-dev \
    && apk add --no-cache  rabbitmq-c-dev openssh librdkafka-dev vim libpng libpng-dev supervisor
# 安装 PHP 扩展
RUN docker-php-ext-install -j$(nproc) intl mysqli pdo_mysql pcntl bcmath zip sockets

# 提取 Swoole 扩展
RUN docker-php-source extract && \
    mkdir /usr/src/php/ext/swoole && \
    curl -sfL https://github.com/swoole/swoole-src/archive/v5.1.0.tar.gz -o swoole.tar.gz && \
    tar xfz swoole.tar.gz --strip-components=1 -C /usr/src/php/ext/swoole && \
    docker-php-ext-configure swoole \
        --enable-mysqlnd \
        --enable-swoole-curl \
        --enable-cares  \
        --enable-swoole-pgsql \
        --enable-sockets \
        --enable-openssl && \
    docker-php-ext-install -j$(nproc) swoole

# 安装和启用其他扩展
RUN pecl install mongodb redis amqp rdkafka && \
    docker-php-ext-enable mongodb redis swoole amqp rdkafka

# 清理不必要的文件
RUN rm -f swoole.tar.gz && rm -rf /tmp/pear












