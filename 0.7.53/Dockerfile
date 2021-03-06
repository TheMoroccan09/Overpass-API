FROM nginx:stable

RUN addgroup overpass && adduser --home /db --disabled-password --gecos overpass --ingroup overpass overpass

RUN apt-get update \
    && apt-get install --no-install-recommends --no-install-suggests -y \
        autoconf \
        automake \
        ca-certificates \
        expat \
        libexpat1-dev \
        g++ \
        liblz4-1 \
        liblz4-dev \
        libtool \
        m4 \
        make \
        zlib1g \
        zlib1g-dev


ADD http://dev.overpass-api.de/releases/osm-3s_v0.7.53.tar.gz /app/src.tar.gz

RUN mkdir -p /app/src \
    && cd /app/src \
    && tar -x -z --strip-components 1 -f ../src.tar.gz \
    && autoscan \
    && aclocal \
    && autoheader \
    && libtoolize \
    && automake --add-missing  \
    && autoconf \
    && CXXFLAGS='-O2' CFLAGS='-O2' ./configure --prefix=/app --enable-lz4 \
    && make -j $(grep -c ^processor /proc/cpuinfo) dist install clean \
    && apt-get remove -y \
        autoconf \
        automake \
        libexpat1-dev \
        g++ \
        libtool \
        liblz4-dev \
        m4 \
        make \
        zlib1g-dev \
    && apt-get autoremove -y \
    && mkdir -p /db/diffs /app/etc \
    && cp -r /app/src/rules /app/etc/rules \
    && rm -rf /app/src /app/src.tar.gz

RUN apt-get install --no-install-recommends --no-install-suggests -y \
        supervisor \
        bash \
        lftp \
        wget \
        fcgiwrap \
        bzip2


RUN mkdir /nginx && chown nginx:nginx /nginx && chown -R overpass:overpass /db
COPY etc/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY etc/nginx-overpass.conf /etc/nginx/nginx.conf
COPY bin/apply_osc_to_db.sh bin/fetch_osc.sh bin/rules_loop.sh bin/dispatcher_start.sh /app/bin/
COPY docker-entrypoint.sh /app/
RUN chmod a+x /app/docker-entrypoint.sh /app/bin/apply_osc_to_db.sh /app/bin/fetch_osc.sh /app/bin/rules_loop.sh /app/bin/dispatcher_start.sh
VOLUME /db

ENV OVERPASS_RULES_LOAD 1
EXPOSE 80
# CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
CMD ["/app/docker-entrypoint.sh"]
