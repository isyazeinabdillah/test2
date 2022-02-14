FROM alpine:latest

ENV NGINX_VERSION 1.20.1
ENV OPENSSL_VERSION 1.1.1a
ENV MAXMIND_VERSION=1.4.2

RUN CONFIG="\
--prefix=/opt/nginx \
        --modules-path=/usr/lib/nginx/modules \
        --with-openssl=/usr/local/src/openssl-$OPENSSL_VERSION \
        --with-http_v2_module --with-http_ssl_module \
        --with-http_stub_status_module \
        --add-module=/usr/local/src/ngx_cache_purge \
        --with-http_dav_module \
        --with-file-aio \
        --with-threads \
        --with-http_geoip_module \
        --with-stream \
        --with-http_mp4_module \
        --with-stream_geoip_module=dynamic \
        --add-module=/usr/local/src/nginx-push-stream-module \
        --add-module=/usr/local/src/ngx_devel_kit \
        --add-module=/usr/local/src/echo-nginx-module \
        --add-module=/usr/local/src/set-misc-nginx-module \
        --add-module=/usr/local/src/headers-more-nginx-module \
        --add-module=/usr/local/src/nginx-module-vts \
#	--add-module=/usr/local/src/ngx_http_geoip2_module \
" \
&& apk add --no-cache --virtual .build-deps \
	gcc \
	libc-dev \
	make \
	openssl-dev \
	pcre-dev \
	zlib-dev \
	linux-headers \
	curl \
	gnupg \
	libxslt-dev \
	gd-dev \
	geoip-dev \
	git \
	tzdata \
&& cat /usr/share/zoneinfo/Asia/Jakarta > /etc/localtime \
&& mkdir -p /usr/local/src \
&& curl -fSL http://nginx.org/download/nginx-$NGINX_VERSION.tar.gz -o nginx.tar.gz \
&& curl -fSL https://www.openssl.org/source/openssl-$OPENSSL_VERSION.tar.gz -o openssl.tar.gz \
&& tar -zxC /usr/local/src -f nginx.tar.gz \
&& tar -zxC /usr/local/src -f openssl.tar.gz \
&& rm -rf nginx.tar.gz openssl.tar.gz \
&& git clone https://github.com/FRiCKLE/ngx_cache_purge.git /usr/local/src/ngx_cache_purge \
&& git clone https://github.com/wandenberg/nginx-push-stream-module.git /usr/local/src/nginx-push-stream-module \
&& git clone https://github.com/openresty/echo-nginx-module.git /usr/local/src/echo-nginx-module \
&& git clone https://github.com/simpl/ngx_devel_kit.git /usr/local/src/ngx_devel_kit \
&& git clone https://github.com/openresty/set-misc-nginx-module.git /usr/local/src/set-misc-nginx-module \
&& git clone https://github.com/openresty/headers-more-nginx-module.git /usr/local/src/headers-more-nginx-module \
&& git clone https://github.com/vozlt/nginx-module-vts.git /usr/local/src/nginx-module-vts \
&& git clone https://github.com/leev/ngx_http_geoip2_module.git /usr/local/src/ngx_http_geoip2_module \
&& cd /usr/local/src/nginx-$NGINX_VERSION \
&& sed -i -e 's/\#define NGINX\_VERSION      \"'$NGINX_VERSION'\"/\#define NGINX\_VERSION      \"'hseo'\"/g' src/core/nginx.h \
&& ./configure $CONFIG \
&& make -j$(getconf _NPROCESSORS_ONLN) \
&& make install \
&& strip /opt/nginx/sbin/nginx* \
&& strip /usr/lib/nginx/modules/*.so \
&& rm -rf /usr/local/src/nginx-$NGINX_VERSION
RUN apk add --no-cache --virtual .gettext gettext \
&& mv /usr/bin/envsubst /tmp/ \
&& runDeps="$( \
	scanelf --needed --nobanner /opt/nginx/sbin/nginx /usr/lib/nginx/modules/*.so /tmp/envsubst \
	| awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
	| sort -u \
	| xargs -r apk info --installed \
	| sort -u \
)" \
&& apk add --no-cache --virtual .nginx-rundeps $runDeps \
&& apk del .build-deps \
&& apk del .gettext \
&& mv /tmp/envsubst /usr/local/bin/
RUN rm -rf /usr/local/src
RUN echo > /etc/sysctl.d/00-alpine.conf
