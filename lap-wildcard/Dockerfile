FROM internetrix/webdev:lap
MAINTAINER Jonathon Su <jonathon.su@internetrix.com.au>
ARG DEBIAN_FRONTEND=noninteractive

### PHP Configuration
# Install xdebug for PHP 5.6+
RUN apt-get -y update
RUN apt-get -y install php-xdebug

# Add custom PHP 5.6 | 7.0 | 7.1 | 7.2 | 7.3 | 7.4 | 8.0 | 8.1 | 8.2 config
ADD config/php/webdev_common_php_config.ini /etc/php/webdev_common_php_config.ini
RUN cat /etc/php/webdev_common_php_config.ini | tee -a /etc/php/5.6/fpm/php.ini /etc/php/7.0/fpm/php.ini /etc/php/7.1/fpm/php.ini /etc/php/7.2/fpm/php.ini /etc/php/7.3/fpm/php.ini /etc/php/7.4/fpm/php.ini /etc/php/8.0/fpm/php.ini /etc/php/8.1/fpm/php.ini /etc/php/8.2/fpm/php.ini

### Apache Configuration
RUN a2enmod rewrite proxy proxy_fcgi proxy_http expires ssl vhost_alias headers env

# Setup Apache config for webdev server
ADD config/apache/conf/webdev.conf /etc/apache2/conf-available/webdev.conf
RUN a2enconf webdev.conf

# Setup /phpinfo alias
RUN mkdir /var/www/php/ && \
	chmod +x /var/www/php/ && \
	echo '<?php phpinfo() ?>' > /var/www/php/phpinfo.php && \
	chmod +x /var/www/php/phpinfo.php && \
	echo 'Alias /phpinfo /var/www/php/phpinfo.php' > /etc/apache2/conf-available/phpinfo.conf && \
 	a2enconf phpinfo.conf

# Setup script to handle project sync to /tmpwww/ folder
ADD rsync_project_to_tmpwww.php /var/www/php/rsync_project_to_tmpwww.php
RUN chmod +x /var/www/php/rsync_project_to_tmpwww.php

# Setup Apache wildcard vhost for multi PHP versions
ADD config/apache/sites/apache-wildcard-php82-vhost.conf /etc/apache2/sites-available/001-wildcard-php82-vhost.conf
ADD config/apache/sites/apache-wildcard-php81-vhost.conf /etc/apache2/sites-available/001-wildcard-php81-vhost.conf
ADD config/apache/sites/apache-wildcard-php80-vhost.conf /etc/apache2/sites-available/001-wildcard-php80-vhost.conf
ADD config/apache/sites/apache-wildcard-php74-vhost.conf /etc/apache2/sites-available/001-wildcard-php74-vhost.conf
ADD config/apache/sites/apache-wildcard-php73-vhost.conf /etc/apache2/sites-available/001-wildcard-php73-vhost.conf
ADD config/apache/sites/apache-wildcard-php72-vhost.conf /etc/apache2/sites-available/001-wildcard-php72-vhost.conf
ADD config/apache/sites/apache-wildcard-php71-vhost.conf /etc/apache2/sites-available/001-wildcard-php71-vhost.conf
ADD config/apache/sites/apache-wildcard-php70-vhost.conf /etc/apache2/sites-available/001-wildcard-php70-vhost.conf
ADD config/apache/sites/apache-wildcard-php56-vhost.conf /etc/apache2/sites-available/002-wildcard-php56-vhost.conf

# Setup Apache wildcard vhost for multi PHP versions
ADD config/apache/sites/apache-wildcard-tmpfs-php70-vhost.conf /etc/apache2/sites-available/003-wildcard-tmpfs-php70-vhost.conf

# Default wildcard has lowest priority
ADD config/apache/sites/apache-wildcard-default-vhost.conf /etc/apache2/sites-available/111-wildcard-default.conf

RUN a2ensite \
    001-wildcard-php82-vhost.conf \
    001-wildcard-php81-vhost.conf \
    001-wildcard-php80-vhost.conf \
    001-wildcard-php74-vhost.conf \
    001-wildcard-php73-vhost.conf \
    001-wildcard-php72-vhost.conf \
    001-wildcard-php71-vhost.conf \
    001-wildcard-php70-vhost.conf \
    002-wildcard-php56-vhost.conf \
    003-wildcard-tmpfs-php70-vhost.conf \
    111-wildcard-default.conf

# Setup Webgrind - web based front-end xdebug profile information presentation tool
ADD config/apache/conf/apache-phpwebgrind.conf /etc/apache2/conf-available/phpwebgrind.conf
RUN mkdir /var/www/php/xdebug.profiler && \
    chmod 777 /var/www/php/xdebug.profiler && \
	cd /var/www/php && \
	git clone https://github.com/jokkedk/webgrind.git webgrind && \
	a2enconf phpwebgrind.conf

# Forward apache logs to docker console.
RUN ln -sf /dev/stdout /var/log/apache2/access.log
RUN ln -sf /dev/stderr /var/log/apache2/error.log

# Run apache in foreground
ADD apache-foreground.sh /usr/local/bin/
RUN ["chmod", "+x", "/usr/local/bin/apache-foreground.sh"]

# Add a template .bashrc so password is asked for SSH only once in session
ADD bashrc-example /root/.bashrc

EXPOSE 80 443

ENV WEBDEV_PHPMYADMIN_DB_HOST 	mariadb
ENV WEBDEV_PHPMYADMIN_DB_USER 	root
ENV WEBDEV_PHPMYADMIN_DB_PW 	root123
ENV WEBDEV_ENABLE_PHP_56_FPM 	1
ENV WEBDEV_ENABLE_PHP_70_FPM 	1
ENV WEBDEV_ENABLE_PHP_71_FPM 	0
ENV WEBDEV_ENABLE_PHP_72_FPM 	1
ENV WEBDEV_ENABLE_PHP_73_FPM 	1
ENV WEBDEV_ENABLE_PHP_74_FPM 	1
ENV WEBDEV_ENABLE_PHP_80_FPM 	1
ENV WEBDEV_ENABLE_PHP_81_FPM 	1
ENV WEBDEV_ENABLE_PHP_82_FPM 	1
ENV XDEBUG_PHP_56_MODE          'develop'
ENV XDEBUG_PHP_70_MODE          'develop'
ENV XDEBUG_PHP_71_MODE          'develop'
ENV XDEBUG_PHP_72_MODE          'develop'
ENV XDEBUG_PHP_73_MODE          'develop'
ENV XDEBUG_PHP_74_MODE          'develop'
ENV XDEBUG_PHP_80_MODE          'develop'
ENV XDEBUG_PHP_81_MODE          'develop'
ENV XDEBUG_PHP_82_MODE          'develop'
ENV WEBDEV_REMOTE_HOST_IP		192.168.99.1
ENV WEBDEV_CIFS_HOST_FOLDER 	//192.168.99.1/www
ENV WEBDEV_CIFS_SMB_VERSION 	3.0
ENV WEBDEV_CIFS_USER 			container
ENV WEBDEV_CIFS_PW 				container123
ENV WEBDEV_POSTFIX_RELAYHOST    [mailhog]:1025
ENV BLACKFIRE_SERVER_ID         ''
ENV BLACKFIRE_SERVER_TOKEN      ''
ENV BLACKFIRE_CLIENT_ID         ''
ENV BLACKFIRE_CLIENT_TOKEN      ''
ENV COMPOSER_DEFAULT_VERSION    2
ENV COMPOSER_GITHUB_TOKEN       ''

CMD ["/usr/local/bin/apache-foreground.sh"]
