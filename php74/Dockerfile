FROM php:7.4-fpm-alpine

RUN sed -i "s/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g" /etc/apk/repositories; \
	apk update; \
	apk add --no-cache \
	bash \
	sed \
	ghostscript \
	imagemagick \
	libzip

RUN set -ex; \
	apk add --no-cache --virtual .build-deps \
		$PHPIZE_DEPS \
		freetype-dev \
		imagemagick-dev \
		libjpeg-turbo-dev \
		libpng-dev \
		libzip-dev \
		libmcrypt-dev \
		gmp-dev \
	; \
	docker-php-ext-configure gd --with-freetype --with-jpeg; \
	docker-php-ext-install -j "$(nproc)" \
		bcmath \
		gd \
		exif \
		pdo_mysql \
		mysqli \
		zip \
		gmp \
	; \
	pecl install imagick-3.4.4 mcrypt redis; \
	docker-php-ext-enable imagick mcrypt redis; \
	\
	runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)"; \
	apk add --virtual .phpexts-rundeps $runDeps; \
	apk del .build-deps

# config
RUN set -eux; \
	mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"; \
	docker-php-ext-enable opcache; \
	{ \
		echo 'opcache.memory_consumption=256'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=10000'; \
		echo 'opcache.revalidate_freq=240'; \
		echo 'opcache.fast_shutdown=1'; \
		echo 'opcache.file_cache=/tmp'; \
	} > /usr/local/etc/php/conf.d/opcache.ini; \
	{ \
		echo 'error_reporting = E_ERROR | E_WARNING | E_PARSE | E_CORE_ERROR | E_CORE_WARNING | E_COMPILE_ERROR | E_COMPILE_WARNING | E_RECOVERABLE_ERROR'; \
		echo 'display_errors = Off'; \
		echo 'display_startup_errors = Off'; \
		echo 'log_errors = On'; \
		echo 'error_log = /dev/stderr'; \
		echo 'log_errors_max_len = 1024'; \
		echo 'ignore_repeated_errors = On'; \
		echo 'ignore_repeated_source = Off'; \
		echo 'html_errors = Off'; \
	} > /usr/local/etc/php/conf.d/error-logging.ini; \
	{ \
		echo 'post_max_size=100M'; \
		echo 'upload_max_filesize=100M'; \
		echo 'max_file_uploads=50'; \
	} > /usr/local/etc/php/conf.d/upload.ini; \
	rm -rf /tmp/*