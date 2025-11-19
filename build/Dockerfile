ARG IMAGE_VERSION
FROM ubuntu:24.04

#ARG DEBIAN_FRONTEND=noninteractive
ENV TZ=Europe/Istanbul 
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

USER root

### Update OS
RUN apt-get -y update && apt-get -y upgrade

### Install Packages
RUN apt-get -y install \
#bind9-dnsutils \
vim \
less \
net-tools \
iputils-ping \
iproute2 \
traceroute \
sudo \
openssh-server \
iptables \
ifupdown \
wireguard \
curl \
lynx \
htop \
build-essential \
supervisor


# NOTE: When updating PHP_VERSION, update the following as well:
# ./conf/supervisor/supervisord.conf
# ./conf/nginx/conf.d/default.conf
# ./php/{php_version}/*

# Install Craft Requirements
RUN set -x \
    && apt-get update \
    && apt-get install -yq --no-install-recommends \
        apt-utils \
        curl \
        gnupg2 \
        iproute2 \
        mysql-client \
        python3-pip \
        python3-setuptools \
        python3-wheel \
        software-properties-common \
        unzip \
        zip \
#    && LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/php -y \
    && curl -o /usr/share/keyrings/nginx_signing.key http://nginx.org/keys/nginx_signing.key \
    && echo "deb [signed-by=/usr/share/keyrings/nginx_signing.key] http://nginx.org/packages/mainline/ubuntu/ noble nginx" > /etc/apt/sources.list.d/nginx.list \
    && apt-get update && apt-get install -yq --no-install-recommends \
        nginx \
        php-fpm \
        php8.3-bcmath \
        php8.3-cli \
        php8.3-curl \
        php8.3-fpm \
        php8.3-gd \
        php8.3-gmp \
        php8.3-intl \
        php8.3-mbstring \
        php8.3-mysql \
        php8.3-opcache \
        php8.3-readline \
        php8.3-soap \
        php8.3-xml \
        php8.3-zip \
        php8.3-imagick \
        php8.3-redis \
    && printf "#!/bin/sh\nexit 0" > /usr/sbin/policy-rc.d \
    && apt-get autoremove --purge -y \
        software-properties-common \
        gnupg2 \
    && apt-get -y clean \
    && rm -rf /var/lib/apt/lists/* /var/tmp/* \
    && sed -i \
        -e "s/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g" \
        -e "s/memory_limit\s*=\s*.*/memory_limit = 256M/g" \
        -e "s/upload_max_filesize\s*=\s*2M/upload_max_filesize = 100M/g" \
        -e "s/post_max_size\s*=\s*8M/post_max_size = 100M/g" \
        -e "s/max_execution_time = 30/max_execution_time = 180/g" \
        -e "s/max_input_time = 60/max_input_time = 180/g" \
        -e "s/variables_order = \"GPCS\"/variables_order = \"EGPCS\"/g" \
        -e "s/;opcache.enable=1/opcache.enable=1/"\
        -e "s/;opcache.memory_consumption=128/opcache.memory_consumption=512/g" \
        -e "s/;opcache.interned_strings_buffer=8/opcache.interned_strings_buffer=64/g" \
        -e "s/;opcache.max_accelerated_files=10000/opcache.max_accelerated_files=30000/g" \
        -e "s/;opcache.revalidate_freq=2/opcache.revalidate_freq=0/g" \
        /etc/php/8.3/fpm/php.ini \
    && sed -i \
        -e "s/;daemonize\s*=\s*yes/daemonize = no/g" \
        /etc/php/8.3/fpm/php-fpm.conf \
    && sed -i \
        -e "s/;catch_workers_output\s*=\s*yes/catch_workers_output = yes/g" \
        -e "s/pm.max_children = 5/pm.max_children = 4/g" \
        -e "s/pm.start_servers = 2/pm.start_servers = 3/g" \
        -e "s/pm.min_spare_servers = 1/pm.min_spare_servers = 2/g" \
        -e "s/pm.max_spare_servers = 3/pm.max_spare_servers = 4/g" \
        -e "s/;pm.max_requests = 500/pm.max_requests = 200/g" \
        -e "s/^;clear_env = no$/clear_env = no/" \
        /etc/php/8.3/fpm/pool.d/www.conf

# Install Composer
RUN curl -o /tmp/composer-setup.php https://getcomposer.org/installer \
  && curl -o /tmp/composer-setup.sig https://composer.github.io/installer.sig \
  && php -r "if (hash('SHA384', file_get_contents('/tmp/composer-setup.php')) !== trim(file_get_contents('/tmp/composer-setup.sig'))) { unlink('/tmp/composer-setup.php'); echo 'Invalid installer' . PHP_EOL; exit(1); }" \
  && php /tmp/composer-setup.php --no-ansi --install-dir=/usr/local/bin --filename=composer  \
  && rm -rf /tmp/*

RUN sed -i 's|listen = /run/php/php8.3-fpm.sock|listen = 0.0.0.0:9000|' /etc/php/8.3/fpm/pool.d/www.conf

RUN mv /usr/sbin/resolvconf /usr/sbin/resolvconf.real

RUN printf "#!/bin/sh\nexit 0\n" > /usr/sbin/resolvconf

RUN chmod +x /usr/sbin/resolvconf

# Nginx config
COPY copyfiles/data-nginx /etc/nginx

# Override default nginx welcome page
COPY copyfiles/data-html /usr/share/nginx/html

### Copy Settins Files For Services
COPY copyfiles/supervisord.conf /etc/supervisor/supervisord.conf
COPY copyfiles/10-wireguard.conf /etc/sysctl.d/10-wireguard.conf

### Start Services
RUN service php8.3-fpm start
RUN service nginx start

### Expose Port For SSH Service
EXPOSE 80
EXPOSE 443

# Running NGINX Service Demon
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]
