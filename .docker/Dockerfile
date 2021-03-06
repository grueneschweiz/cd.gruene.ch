FROM php:7.4-apache

# update the repository sources list
# and install basic dependencies
RUN apt-get update -y \
  && apt-get install -y \
    openssl \
    zip \
    unzip \
    git \
    vim \
    less \
    wget

# install & enable mysql driver
RUN docker-php-ext-install pdo_mysql

# install & enable xdebug
RUN pecl install xdebug
RUN docker-php-ext-enable xdebug

# install imagick
# RUN apt-get install -y libmagickwand-dev --no-install-recommends
# RUN pecl install imagick
# RUN docker-php-ext-enable imagick
# normally thats it. but we have to us a custom build, since imagemagick
# 7.0 is not in the repos yet. So here comes the custom build:
WORKDIR /tmp
RUN apt-get -y install libpng-dev libjpeg-dev libgs-dev libbz2-dev libfreetype6-dev libtiff-dev librsvg2-dev libxml2-dev
RUN wget https://github.com/ImageMagick/ImageMagick/archive/7.0.10-34.tar.gz
RUN tar -xvf 7.0.10-34.tar.gz
WORKDIR /tmp/ImageMagick-7.0.10-34
RUN ./configure --with-freetype --with-jpeg --with-png --with-rsvg
RUN make
RUN make install

WORKDIR /tmp
RUN wget https://pecl.php.net/get/imagick
RUN mkdir imagick-latest
RUN tar -xvf imagick -C /tmp/imagick-latest --strip-components=1
WORKDIR /tmp/imagick-latest
RUN phpize
RUN ./configure --with-imagick=/opt/local
RUN make
RUN make install

RUN docker-php-ext-enable imagick

WORKDIR /var/www/html


# install intl
RUN apt-get install -y libicu-dev
RUN docker-php-ext-configure intl
RUN docker-php-ext-install intl

# change uid and gid of apache to docker user uid/gid
RUN usermod -u 1000 www-data && groupmod -g 1000 www-data

# reconfigure the virtual hosts to use the ones defined in .docker/apache-vhost.conf
COPY .docker/apache-vhost.conf /etc/apache2/sites-available/custom-vhost.conf
RUN a2dissite 000-default.conf
RUN a2ensite custom-vhost.conf

# use custom php.ini
COPY .docker/php.ini /usr/local/etc/php/

# activate mod rewrite
RUN a2enmod rewrite

# set our application folder as an environment variable
ENV APP_HOME /var/www/html

# set the working dir
WORKDIR $APP_HOME

# change ownership of our application
RUN chown -R www-data:www-data $APP_HOME
