FROM nginx:1.23.2-alpine AS builder

# nginx:alpine contains NGINX_VERSION environment variable, like so:
ENV NGINX_VERSION 1.23.2
ENV OPENSSL_VERSION="1.1.1i"

# For latest build deps, see https://github.com/nginxinc/docker-nginx/blob/master/mainline/alpine/Dockerfile
RUN apk add --no-cache --virtual .build-deps \
  git \
  gcc \
  libc-dev \
  make \
  # openssl-dev \
  pcre2-dev \
  zlib-dev \
  linux-headers \
  libxslt-dev \
  gd-dev \
  geoip-dev \
  libedit-dev \
  bash \
  alpine-sdk \
  findutils

# Download and configure OpenSSL sources
RUN set -x \
  && mkdir /opt/local \
  && export LD_LIBRARY_PATH=/opt/local/lib \
  && wget --no-check-certificate -O /tmp/openssl-${OPENSSL_VERSION}.tar.gz "https://www.openssl.org/source/openssl-${OPENSSL_VERSION}.tar.gz" \
  && tar -xvf /tmp/openssl-${OPENSSL_VERSION}.tar.gz --directory /opt/local \
  && rm -rf /tmp/openssl-${OPENSSL_VERSION}.tar.gz \
  && cd /opt/local/openssl-${OPENSSL_VERSION} \
  && ./config --prefix=/opt/local --openssldir=/opt/local/ssl enable-ssl3 enable-ssl3-method enable-weak-ssl-ciphers

# Set the workdir so the make commands work
WORKDIR /opt/local/openssl-${OPENSSL_VERSION}

# Build OpenSSL from sources
RUN make \
  && make install

# Cleanup after OpenSSL build
RUN cd .. \
  && rm -rf openssl-${OPENSSL_VERSION}

# Enable old ciphers (e.g. 'RC4-SHA' & 'RC4-MD5')
RUN echo "openssl_conf = default_conf"|cat - /opt/local/ssl/openssl.cnf > /tmp/out && mv /tmp/out /opt/local/ssl/openssl.cnf
RUN echo "" >> /opt/local/ssl/openssl.cnf
RUN echo "[default_conf]" >> /opt/local/ssl/openssl.cnf
RUN echo "ssl_conf = ssl_sect" >> /opt/local/ssl/openssl.cnf
RUN echo "" >> /opt/local/ssl/openssl.cnf
RUN echo "[ssl_sect]" >> /opt/local/ssl/openssl.cnf
RUN echo "system_default = system_default_sect" >> /opt/local/ssl/openssl.cnf
RUN echo "" >> /opt/local/ssl/openssl.cnf
RUN echo "[system_default_sect]" >> /opt/local/ssl/openssl.cnf
RUN echo "CipherString = ALL:@SECLEVEL=0" >> /opt/local/ssl/openssl.cnf

# Set path and lib path
ENV PATH /opt/local/bin:$PATH
ENV LD_LIBRARY_PATH /opt/local/lib:$LD_LIBRARY_PATH

WORKDIR /

# Download sources
RUN wget "http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz" -O nginx.tar.gz

# Reuse same cli arguments as the nginx:alpine image used to build (but remove the erroneous -fomit-frame-pointer)
RUN mkdir -p /usr/src && \
    tar -zxC /usr/src -f nginx.tar.gz && \
    cd /usr/src/nginx-$NGINX_VERSION && \
    ./configure --with-http_ssl_module && \
    make && make install

FROM nginx:1.23.2-alpine

# Copy OpenSSL
COPY --from=builder /opt/local /opt/local
# Copy new nginx binary
COPY --from=builder /usr/sbin/nginx /usr/sbin/nginx

# Set path and lib path
ENV PATH /opt/local/bin:$PATH
ENV LD_LIBRARY_PATH /opt/local/lib:$LD_LIBRARY_PATH

RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
EXPOSE 443
STOPSIGNAL SIGTERM
CMD ["nginx", "-g", "daemon off;"]
