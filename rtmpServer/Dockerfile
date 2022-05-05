ARG NGINX_VERSION=1.9.9
ARG NGINX_RTMP_VERSION=1.2.1

FROM alpine:3.15.4 AS build
ARG NGINX_VERSION
ARG NGINX_RTMP_VERSION

COPY nginx-rtmp-fallthrough-fix.patch /tmp/nginx-rtmp-fallthrough-fix.patch

RUN \
  mkdir -p /var/nginx/rec && mkdir -p /var/nginx/logs && \
  adduser nginx --system --no-create-home --group --disabled-login --disabled-password && \
  build_pkgs="build-base linux-headers openssl-dev pcre-dev wget zlib-dev patch" && \
  apk --no-cache add ${build_pkgs} ${runtime_pkgs} && \
  cd /tmp && \
  wget https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
  wget wget https://github.com/defanator/nginx-rtmp-module/archive/fix-build-with-gcc8.zip && \
  tar xzf nginx-${NGINX_VERSION}.tar.gz && \
  unzip fix-build-with-gcc8.zip && \
  cd /tmp/nginx-${NGINX_VERSION} && \
  ./configure  --add-module=../nginx-rtmp-module-fix-build-with-gcc8 && \
  make && \
  make install && \
  rm -rf /tmp/* && \
  apk del ${build_pkgs} && \
  rm -rf /var/cache/apk/* && \
  rc-update add nginx

FROM alpine:3.15.4

LABEL MAINTAINER Pedro Freitas <1pedrohfreitas@gmail.com>

COPY --from=build /usr/sbin/nginx /usr/sbin/nginx
COPY --from=build /etc/nginx /etc/nginx
COPY nginx.conf /etc/nginx/nginx.conf

RUN \
  apk add --no-cache ca-certificates openssl pcre zlib tzdata git && \
  mkdir /var/log/nginx && \
  ln -sf /dev/stdout /var/log/nginx/access.log && \
  ln -sf /dev/stderr /var/log/nginx/error.log

EXPOSE 1935

CMD ["nginx"]