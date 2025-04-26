# syntax=docker/dockerfile:1.10
FROM alpine:3.21

# This MUST be one of the FIPS validated versions listed at
# https://openssl-library.org/source/
ARG OPENSSL_VERSION=3.1.2

# download source for supplied version then build and install the FIPS module.
RUN apk --no-cache \
        --virtual .build-deps \
        add make gcc libgcc musl-dev linux-headers perl \
    && mkdir /work && cd /work \
    && wget https://github.com/openssl/openssl/releases/download/openssl-${OPENSSL_VERSION}/openssl-${OPENSSL_VERSION}.tar.gz \
    && wget https://github.com/openssl/openssl/releases/download/openssl-${OPENSSL_VERSION}/openssl-${OPENSSL_VERSION}.tar.gz.sha256 \
    && echo $(cat openssl-${OPENSSL_VERSION}.tar.gz.sha256) openssl-${OPENSSL_VERSION}.tar.gz | sha256sum -c - \
    && tar -xf openssl-${OPENSSL_VERSION}.tar.gz \
    && cd openssl-${OPENSSL_VERSION} \
    && ./Configure enable-fips --libdir=lib --prefix=/usr \
    && make -j 4 install_fips \
    && cd / && rm -fr /work \
    && apk del .build-deps

# install openssl and then generate the fipsmodule.cnf configuration file from
# the built FIPS module. Modify the distribution openssl.cnf file to activate
# and use the built FIPS module. A couple sanity checks are run at the end to
# ensure the sha1 works but md5 doesn't work
RUN apk --no-cache -U add openssl \
    && openssl fipsinstall \
        -pedantic \
        -module /usr/lib/ossl-modules/fips.so \
        -out /etc/ssl/fipsmodule.cnf \
    && sed -i 's|# .include fipsmodule.cnf|.include /etc/ssl/fipsmodule.cnf|' /etc/ssl/openssl.cnf \
    && sed -i 's|providers = providers_sect|providers = providers_sect\nalg_section = algorithm_sect|' /etc/ssl/openssl.cnf \
    && sed -i 's|# fips = fips_sect|fips = fips_sect\nbase = base_sect|' /etc/ssl/openssl.cnf \
    && echo '' >> /etc/ssl/openssl.cnf \
    && echo '[base_sect]' >> /etc/ssl/openssl.cnf \
    && echo 'activate = 1' >> /etc/ssl/openssl.cnf \
    && echo '' >> /etc/ssl/openssl.cnf \
    && echo '[algorithm_sect]' >> /etc/ssl/openssl.cnf \
    && echo 'default_properties = fips=yes' >> /etc/ssl/openssl.cnf \
    && ! openssl md5 /etc/ssl/openssl.cnf \
    && openssl sha1 /etc/ssl/openssl.cnf \
    && openssl list -provider-path providers -provider fips -providers
