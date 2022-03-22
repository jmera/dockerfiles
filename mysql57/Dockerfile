FROM debian:buster-slim

RUN groupadd -r mysql && useradd -r -g mysql mysql

RUN apt-get update && apt-get install -y --no-install-recommends gnupg dirmngr && rm -rf /var/lib/apt/lists/*

# add gosu for easy step-down from root
# https://github.com/tianon/gosu/releases
ENV GOSU_VERSION 1.14
RUN set -eux; \
  savedAptMark="$(apt-mark showmanual)"; \
  apt-get update; \
  apt-get install -y --no-install-recommends ca-certificates wget; \
  rm -rf /var/lib/apt/lists/*; \
  dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')"; \
  wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch"; \
  wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch.asc"; \
  export GNUPGHOME="$(mktemp -d)"; \
  gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4; \
  gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu; \
  gpgconf --kill all; \
  rm -rf "$GNUPGHOME" /usr/local/bin/gosu.asc; \
  apt-mark auto '.*' > /dev/null; \
  [ -z "$savedAptMark" ] || apt-mark manual $savedAptMark > /dev/null; \
  apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
  chmod +x /usr/local/bin/gosu; \
  gosu --version; \
  gosu nobody true

RUN mkdir /docker-entrypoint-initdb.d

RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends \
    openssl \
# FATAL ERROR: please install the following Perl modules before executing /usr/local/mysql/scripts/mysql_install_db:
# File::Basename
# File::Copy
# Sys::Hostname
# Data::Dumper
    perl \
    xz-utils \
    zstd \
  ; \
  rm -rf /var/lib/apt/lists/*

ENV MYSQL_MAJOR 5.7

RUN set -eux; \
  apt-get update; \
  apt-get install --yes --no-install-recommends \
    bison \
    build-essential \
    ca-certificates \
    cmake \
    gcc \
    git \
    libaio-dev \
    libncurses-dev \
    libssl-dev \
    make \
    pkg-config; \
  git clone --branch=$MYSQL_MAJOR --depth=1 https://github.com/mysql/mysql-server.git /tmp/mysql-server/; \
  mkdir /tmp/mysql-server/build/; \
  cd /tmp/mysql-server/build/; \
  cmake .. \
    -DBUILD_CONFIG=mysql_release \
    -DDOWNLOAD_BOOST=1 \
    -DINSTALL_MYSQLTESTDIR="" \
    -DINSTALL_SBINDIR=/usr/sbin \
    -DSYSCONFDIR=/etc/mysql/ \
    -DWITH_BOOST=/tmp/boost/; \
  make; \
  make install; \
  mkdir -p /etc/mysql/conf.d/; \
  echo '[mysqld]\nskip-host-cache\nskip-name-resolve' > /etc/mysql/conf.d/docker.cnf; \
  rm -rf /var/lib/mysql; \
  mkdir -p /var/lib/mysql /var/run/mysqld; \
  chown -R mysql:mysql /var/lib/mysql /var/run/mysqld; \
  chmod 750 /var/run/mysqld /var/lib/mysql; \
  apt-get remove --yes \
    cmake \
    gcc \
    git \
    make; \
  rm -rf /var/lib/apt/lists/* /tmp/*

VOLUME /var/lib/mysql

COPY docker-entrypoint.sh /usr/local/bin/
RUN ln -s usr/local/bin/docker-entrypoint.sh /entrypoint.sh # backwards compat
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 3306 33060
CMD ["mysqld"]
