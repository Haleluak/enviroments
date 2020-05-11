FROM ubuntu:16.04

MAINTAINER Khanh Nguyen <khanhnv.uit@gmail.com>

ENV LC_ALL=C.UTF-8

RUN apt-get update && \
	apt-get -y install wget \
	curl \
	git \
	software-properties-common \
	python-software-properties \
	debian-archive-keyring \
	mcrypt

RUN add-apt-repository ppa:ondrej/php
RUN apt-get update && apt-get -y install \
	vim \
	python \
    python-dev \
    ca-certificates \
    dialog \
    gcc \
    musl-dev \
    libffi-dev \
    python-pip \
    zip \
    unzip \
    iputils-ping \
    build-essential nghttp2 libnghttp2-dev libssl-dev re2c

# Build openssl 1.1.0g
RUN wget https://www.openssl.org/source/openssl-1.1.0g.tar.gz && \
	tar xzvf openssl-1.1.0g.tar.gz && \
	cd openssl-1.1.0g && \
	./config -Wl,--enable-new-dtags,-rpath,'$(LIBRPATH)' && \
	make && make install && \
	cd ../ && rm -rf openssl-1.1.0g.tar.gz openssl-1.1.0g

# Build Curl with enable nghttp2
RUN wget https://curl.haxx.se/download/curl-7.55.1.tar.bz2 && \
	tar -xvjf curl-7.55.1.tar.bz2 && \
	cd curl-7.55.1 && \
	./configure --with-nghttp2 --prefix=/usr/local --with-ssl && \
	make && make install && ldconfig && \
	cd ../ && rm -rf curl-7.55.1.tar.bz2 curl-7.55.1

# Install php7.2
RUN apt-get update && apt-get -y install php7.2 php7.2-cli php7.2-fpm php7.2-cgi php7.2-common php7.2-dev php7.2-gd \
	php7.2-json php7.2-ldap php7.2-mysql php7.2-odbc php7.2-opcache php7.2-pspell php7.2-readline php7.2-recode \
	php7.2-sqlite3 php7.2-xml libphp7.2-embed php7.2-bcmath php7.2-bz2 php7.2-fpm php7.2-imap php7.2-intl \
	php7.2-mbstring php7.2-xsl php7.2-zip php7.2-dba php-redis php7.2-mongodb php-pear php-sodium

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin/ && mv /usr/bin/composer.phar /usr/bin/composer

# Install composer speed
RUN composer global require hirak/prestissimo
RUN composer global require "fxp/composer-asset-plugin:~1.3"

# Install Nodejs
RUN curl -sL https://deb.nodesource.com/setup_8.x -o nodesource_setup.sh
RUN chmod a+x nodesource_setup.sh && ./nodesource_setup.sh
RUN apt-get update && apt-get -y install nodejs

# Install Supervisor
RUN apt-get update
RUN apt-get -y install supervisor \
	telnet \
	net-tools
RUN mkdir -p /var/log/cache

# Install Phalcon
RUN curl -s https://packagecloud.io/install/repositories/phalcon/stable/script.deb.sh | bash
RUN apt-get update && apt-get install php7.2-phalcon
RUN mkdir -p /vendor \
    && composer --working-dir=/vendor require phalcon/devtools \
    && ln -s /vendor/vendor/phalcon/devtools/phalcon.php /usr/local/bin/phalcon \
    && chmod ugo+x /usr/local/bin/phalcon

# Install Yarn
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update && apt-get install yarn

ADD ./php/www.conf /etc/php/7.2/fpm/pool.d/www.conf
ADD ./php/php.ini /etc/php/7.2/fpm/php.ini
ADD ./php/php-fpm.conf /etc/php/7.2/fpm/php-fpm.conf
ADD ./supervisor/supervisord.conf /etc/supervisor/supervisord.conf


EXPOSE 9000

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]