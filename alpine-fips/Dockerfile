# syntax=docker/dockerfile:1.10
# version of Alpine
ARG ALPINE_VERSION=3.22

# build stage
FROM docker.io/library/alpine:${ALPINE_VERSION} AS builder

# This MUST be one of the FIPS validated versions listed at
# https://openssl-library.org/source/
ARG OPENSSL_VERSION=3.1.2

# download source for the supplied version, verify checksum and
# PGP signature, then build and install the FIPS module.
RUN apk add build-base linux-headers perl gpg gpg-agent \
# do build under /build \
    && mkdir /build && cd /build \
# download tarball, sha, signature \
    && OPENSSL_TAR=openssl-${OPENSSL_VERSION}.tar.gz \
    && URL="https://github.com/openssl/openssl/releases/download/openssl-${OPENSSL_VERSION}" \
    && wget --tries=3 ${URL}/${OPENSSL_TAR} \
    && wget --tries=3 ${URL}/${OPENSSL_TAR}.sha256 \
    && wget --tries=3 ${URL}/${OPENSSL_TAR}.asc \
# verify the sha \
    && echo $(cat ${OPENSSL_TAR}.sha256) ${OPENSSL_TAR} | sha256sum -c - \
# download signature keys and verify the signature \
    && gpg --recv-keys \
        'BA54 73A2 B058 7B07 FB27 CF2D 2160 94DF D0CB 81EF' \
        '7953 AC1F BC3D C8B3 B292 393E D5E9 E43F 7DF9 EE8C' \
        'EFC0 A467 D613 CB83 C7ED 6D30 D894 E2CE 8B3D 79F5' \
        'A21F AB74 B008 8AA3 6115 2586 B8EF 1A6B A9DA 2D5C' \
        'E5E5 2560 DD91 C556 DDBD A5D0 2064 C536 41C2 5E5D' \
    && gpg --verify ${OPENSSL_TAR}.asc ${OPENSSL_TAR} \
# extract, configure, build \
    && tar -xf ${OPENSSL_TAR} \
    && cd openssl-${OPENSSL_VERSION} \
    && ./Configure enable-fips --libdir=lib --prefix=/usr \
# install the FIPS module \
    && make -j $(nproc) install_fips \
        || { echo "OpenSSL build failed"; exit 1; } \
    && test -f /usr/lib/ossl-modules/fips.so

# final stage
FROM docker.io/library/alpine:${ALPINE_VERSION}

LABEL maintainer="David Crowley <david.i.crowley@jpl.nasa.gov>"
LABEL version="alpine${ALPINE_VERSION}-ossl${OPENSSL_VERSION}-fips"
LABEL description="Alpine ${ALPINE_VERSION} with FIPS-compliant OpenSSL ${OPENSSL_VERSION}"

# copy the built FIPS .so from the builder stage
COPY --from=builder /usr/lib/ossl-modules/fips.so /usr/lib/ossl-modules/fips.so

# install openssl and then generate the fipsmodule.cnf configuration file from
# the built FIPS module. Modify the distribution openssl.cnf file to activate
# and use the built FIPS module.
RUN apk --no-cache add openssl \
# this eval sets OPENSSLDIR and MODULESDIR \
    && eval $(openssl version -d -m | sed 's/: /=/') \
    && FIPSCNF=${OPENSSLDIR}/fipsmodule.cnf \
    && OPENCNF=${OPENSSLDIR}/openssl.cnf \
    && test -f ${OPENCNF} \
        || { echo "openssl.cnf not found"; exit 1; } \
    && cp ${OPENCNF} ${OPENCNF}.orig \
# make sure our fips module was copied to the correct location \
    && openssl fipsinstall \
        -module ${MODULESDIR}/fips.so \
        -pedantic -out ${FIPSCNF} \
    && test -f ${OPENCNF} \
        || { echo "fipsmodule.cnf not found"; exit 1; } \
# edit the shipped openssl.cnf to enable the FIPS module \
    && sed -i \
        -e "s|# .include fipsmodule.cnf|.include ${FIPSCNF}|" \
        -e "s|# fips = fips_sect|fips = fips_sect\nbase = base_sect|" \
        -e "/providers = provider_sect/a alg_section = algorithm_sect" \
        ${OPENCNF} \
# add the [base_sect] and [algorithm_sect] sections to the end \
    && grep -q "\[base_sect\]" ${OPENCNF} \
        || echo -e "\n[base_sect]\nactivate = 1\n\n[algorithm_sect]\ndefault_properties = fips=yes" >> ${OPENCNF} \
    && chmod 644 ${OPENCNF} ${FIPSCNF} \
# Run some sanity checks on the build, configuration \
    && openssl list -providers | grep FIPS \
        || { echo "FIPS provider not loaded"; exit 1; } \
# The MD5 test should evaluate to false because FIPS doesn't allow MD5 \
    && (! openssl md5 ${OPENCNF} \
        || { echo "MD5 test failed unexpectedly"; exit 1; }) \
    && openssl sha1 ${OPENCNF} \
        || { echo "SHA-1 test failed"; exit 1; } \
    && openssl aes-256-cbc -k "$RANDOM" -in ${OPENCNF} -out /dev/null \
        || { echo "AES test failed"; exit 1; } \
    && openssl list -providers
