FROM ubuntu:18.04

LABEL maintainer="Chris Fidao"

ARG USERNAME=vessel
ARG USER_UID=1000
ARG USER_GID=1337

RUN useradd -ms /bin/bash -u ${USER_GID} ${USERNAME}
WORKDIR /var/www/html

ENV TZ=UTC
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN set -x \
  && apt-get update && apt-get install -y gnupg gosu \
  && gosu nobody true

RUN echo "deb http://ppa.launchpad.net/ondrej/php/ubuntu bionic main" > /etc/apt/sources.list.d/ppa_ondrej_php.list \
  && echo "deb http://ppa.launchpad.net/nginx/development/ubuntu bionic main" > /etc/apt/sources.list.d/ppa_nginx_mainline.list \
  && apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys E5267A6C \
  && apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C300EE8C \
  && apt-get update \
  && apt-get install -y curl zip unzip git supervisor sqlite3 zsh \
  && apt-get install -y nginx php7.3-fpm php7.3-cli \
  php7.3-pgsql php7.3-sqlite3 php7.3-gd \
  php7.3-curl php7.3-memcached \
  php7.3-imap php7.3-mysql php7.3-mbstring \
  php7.3-xml php7.3-zip php7.3-bcmath php7.3-soap \
  php7.3-intl php7.3-readline php7.3-xdebug \
  php7.3-msgpack php7.3-igbinary php7.3-ldap \
  php-redis libnginx-mod-http-uploadprogress \
  && php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
  && mkdir /run/php \
  && apt-get -y autoremove \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
  && sed -i "s/pm\.max_children = .*/pm.max_children = 20/" /etc/php/7.3/fpm/pool.d/www.conf \
  && sed -i "s/pm\.start_servers = .*/pm.start_servers = 10/" /etc/php/7.3/fpm/pool.d/www.conf \
  && sed -i "s/pm\.min_spare_servers = .*/pm.min_spare_servers = 5/" /etc/php/7.3/fpm/pool.d/www.conf \
  && sed -i "s/pm\.max_spare_servers = .*/pm.max_spare_servers = 10/" /etc/php/7.3/fpm/pool.d/www.conf \
  && echo "daemon off;" >> /etc/nginx/nginx.conf

RUN apt-get update \
  #
  # Install Docker CE CLI
  && apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common lsb-release \
  && curl -fsSL https://download.docker.com/linux/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/gpg | apt-key add - 2>/dev/null \
  && add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/$(lsb_release -is | tr '[:upper:]' '[:lower:]') $(lsb_release -cs) stable" \
  && apt-get update \
  && apt-get install -y docker-ce-cli \
  #
  # Install Docker Compose
  && curl -sSL "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose \
  && chmod +x /usr/local/bin/docker-compose

COPY --from=drush/drush /usr/local/bin/drush /usr/local/bin/drush

RUN ln -sf /dev/stdout /var/log/nginx/access.log \
  && ln -sf /dev/stderr /var/log/nginx/error.log

# COPY h5bp /etc/nginx/h5bp
# COPY default /etc/nginx/sites-available/default
# COPY php-fpm.conf /etc/php/7.3/fpm/php-fpm.conf
# COPY xdebug.ini /etc/php/7.3/mods-available/xdebug.ini
# COPY vessel.ini /etc/php/7.3/fpm/conf.d/99-vessel.ini

# EXPOSE 80

# COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
# COPY start-container /usr/local/bin/start-container
RUN chmod +x /usr/local/bin/start-container

RUN chsh -s /bin/zsh

USER $USERNAME

RUN sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

USER root

ENTRYPOINT ["start-container"]
