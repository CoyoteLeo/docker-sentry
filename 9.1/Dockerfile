FROM python:2.7.18-slim-buster

# add our user and group first to make sure their IDs get assigned consistently
RUN groupadd -r sentry && useradd -r -m -g sentry sentry

RUN apt-get update && apt-get install -y --no-install-recommends \
        gcc \
        git \
        libffi-dev \
        libjpeg-dev \
        libmaxminddb-dev \
        libpq-dev \
        libxml2-dev \
        libxmlsec1-dev \
        libxslt-dev \
        libyaml-dev \
        pkg-config \
    && rm -rf /var/lib/apt/lists/*

# Sane defaults for pip
ENV PIP_NO_CACHE_DIR off
ENV PIP_DISABLE_PIP_VERSION_CHECK on

# grab gosu for easy step-down from root
RUN set -x \
    && export GOSU_VERSION=1.11 \
    && fetchDeps=" \
        dirmngr \
        gnupg \
        wget \
    " \
    && apt-get update && apt-get install -y --no-install-recommends $fetchDeps && rm -rf /var/lib/apt/lists/* \
    && wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
    && chmod +x /usr/local/bin/gosu \
    && gosu nobody true \
    && apt-get purge -y --auto-remove $fetchDeps

# grab tini for signal processing and zombie killing
RUN set -x \
    && export TINI_VERSION=0.18.0 \
    && fetchDeps=" \
        dirmngr \
        gnupg \
        wget \
    " \
    && apt-get update && apt-get install -y --no-install-recommends $fetchDeps && rm -rf /var/lib/apt/lists/* \
    && wget -O /usr/local/bin/tini "https://github.com/krallin/tini/releases/download/v$TINI_VERSION/tini" \
    && chmod +x /usr/local/bin/tini \
    && tini -h \
&& apt-get purge -y --auto-remove $fetchDeps

# Support for RabbitMQ and GeoIP
RUN set -x \
    && apt-get update && apt-get install -y --no-install-recommends make && rm -rf /var/lib/apt/lists/* \
    && pip install librabbitmq==1.6.1 maxminddb==1.4.1 \
    && python -c 'import librabbitmq' \
    # Fully verify that the C extension is correctly installed, it unfortunately
    # requires a full check into maxminddb.extension.Reader
    && python -c 'import maxminddb.extension; maxminddb.extension.Reader' \
    && apt-get purge -y --auto-remove make

ARG SENTRY_PLUGINS_VERSION=9.1.2
ARG SENTRY_VERSION=9.1.3

RUN set -x \
    && buildDeps=" \
        g++ \
        dirmngr \
        gnupg \
        wget \
    " \
    && apt-get update && apt-get install -y --no-install-recommends $buildDeps && rm -rf /var/lib/apt/lists/* \
    && mkdir -p /usr/src/sentry \
    && wget -O /usr/src/sentry/sentry-${SENTRY_VERSION}-py27-none-any.whl "https://github.com/CoyoteLeo/sentry/releases/download/${SENTRY_VERSION}/sentry-${SENTRY_VERSION}-py27-none-any.whl" \
    && wget -O /usr/src/sentry/sentry_plugins-${SENTRY_PLUGINS_VERSION}-py2.py3-none-any.whl "https://github.com/getsentry/sentry/releases/download/${SENTRY_PLUGINS_VERSION}/sentry_plugins-${SENTRY_PLUGINS_VERSION}-py2.py3-none-any.whl" \
    && pip install \
        /usr/src/sentry/sentry-${SENTRY_VERSION}-py27-none-any.whl \
        /usr/src/sentry/sentry_plugins-${SENTRY_PLUGINS_VERSION}-py2.py3-none-any.whl \
    && sentry --help \
    && sentry plugins list \
    && rm -r /usr/src/sentry \
    && apt-get purge -y --auto-remove $buildDeps

ENV SENTRY_CONF=/etc/sentry \
    SENTRY_FILESTORE_DIR=/var/lib/sentry/files

RUN mkdir -p $SENTRY_CONF && mkdir -p $SENTRY_FILESTORE_DIR

COPY sentry.conf.py /etc/sentry/
COPY config.yml /etc/sentry/

COPY docker-entrypoint.sh /entrypoint.sh

EXPOSE 9000
VOLUME /var/lib/sentry/files

ENTRYPOINT ["/entrypoint.sh"]
CMD ["run", "web"]
