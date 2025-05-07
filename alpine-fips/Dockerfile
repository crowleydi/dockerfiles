# syntax=docker/dockerfile:1.10

# version of Alpine
ARG BASE_VERSION=3.21
FROM docker.io/library/alpine:${BASE_VERSION}

# This MUST be one of the FIPS validated versions listed at
# https://openssl-library.org/source/
ARG OPENSSL_VERSION=3.1.2

# download source for the supplied version, verify checksum and
# PGP signature, then build and install the FIPS module.
RUN apk --no-cache \
        --virtual .build-deps \
        add \
			make gcc libgcc musl-dev linux-headers \
			perl gpg gpg-agent \
# do build under /work \
    && mkdir /work && cd /work \
# download tarball, sha, signature \
    && OPENSSL_TAR=openssl-${OPENSSL_VERSION}.tar.gz \
    && URL="https://github.com/openssl/openssl/releases/download/openssl-${OPENSSL_VERSION}" \
    && wget ${URL}/${OPENSSL_TAR} \
    && wget ${URL}/${OPENSSL_TAR}.sha256 \
    && wget ${URL}/${OPENSSL_TAR}.asc \
# verify the sha \
    && echo $(cat ${OPENSSL_TAR}.sha256) ${OPENSSL_TAR} | sha256sum -c - \
# download signature keys and verify the signature \
    && gpg --recv-keys \
         BA5473A2B0587B07FB27CF2D216094DFD0CB81EF \
         EFC0A467D613CB83C7ED6D30D894E2CE8B3D79F5 \
    && gpg --verify ${OPENSSL_TAR}.asc ${OPENSSL_TAR} \
# extract, configure, build \
    && tar -xf ${OPENSSL_TAR} \
    && cd openssl-${OPENSSL_VERSION} \
    && ./Configure enable-fips --libdir=lib --prefix=/usr \
# install the FIPS module \
    && make -j 8 install_fips \
# clean up build deps and work area \
    && apk del .build-deps \
    && cd / && rm -fr /work ~/.gnupg

# install openssl and then generate the fipsmodule.cnf configuration file from
# the built FIPS module. Modify the distribution openssl.cnf file to activate
# and use the built FIPS module.
RUN apk --no-cache -U add openssl \
    && FIPSCNF=/etc/ssl/fipsmodule.cnf \
    && OPENCNF=/etc/ssl/openssl.cnf \
    && openssl fipsinstall \
        -pedantic \
        -module /usr/lib/ossl-modules/fips.so \
        -out ${FIPSCNF} \
# edit the shipped openssl.cnf to enable the FIPS module \
    && sed -i "s|# .include fipsmodule.cnf|.include ${FIPSCNF}|" ${OPENCNF} \
    && sed -i "s|# fips = fips_sect|fips = fips_sect\nbase = base_sect|" ${OPENCNF} \
    && sed -i "s|providers = providers_sect|a\\\\alg_section = algorithm_sect|" ${OPENCNF} \
    && echo '' >> ${OPENCNF} \
    && echo '[base_sect]' >> ${OPENCNF} \
    && echo 'activate = 1' >> ${OPENCNF} \
    && echo '' >> ${OPENCNF} \
    && echo '[algorithm_sect]' >> ${OPENCNF} \
    && echo 'default_properties = fips=yes' >> ${OPENCNF} \
# Run some sanity checks on the build, configuration \
# this first test is expected to fail because FIPS doesn't allow MD5 \
    && ! openssl md5 ${OPENCNF} \
    && openssl sha1 ${OPENCNF} \
    && openssl list -providers
