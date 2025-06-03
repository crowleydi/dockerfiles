# syntax=docker/dockerfile:1.10
# version of Alpine
ARG BASE_VERSION=3.21
FROM docker.io/library/alpine:${BASE_VERSION}

# This MUST be one of the FIPS validated versions listed at
# https://openssl-library.org/source/
ARG OPENSSL_VERSION=3.1.2

# download source for the supplied version, verify checksum and
# PGP signature, then build and install the FIPS module.
RUN apk --no-cache --virtual build-deps add \
        build-base linux-headers perl gpg gpg-agent \
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
        'BA54 73A2 B058 7B07 FB27 CF2D 2160 94DF D0CB 81EF' \
        'EFC0 A467 D613 CB83 C7ED 6D30 D894 E2CE 8B3D 79F5' \
        '95A9 908D DFA1 6830 BE9F B900 3D30 A3A9 FF13 60DC' \
        'E5E5 2560 DD91 C556 DDBD A5D0 2064 C536 41C2 5E5D' \
        '8657 ABB2 60F0 56B1 E519 0839 D9C4 D26D 0E60 4491' \
        'B7C1 C143 60F3 53A3 6862 E4D5 231C 84CD DCC6 9C45' \
        '7953 AC1F BC3D C8B3 B292 393E D5E9 E43F 7DF9 EE8C' \
        'C1F3 3DD8 CE1D 4CC6 13AF 14DA 9195 C482 41FB F7DD' \
        'A21F AB74 B008 8AA3 6115 2586 B8EF 1A6B A9DA 2D5C' \
    && gpg --verify ${OPENSSL_TAR}.asc ${OPENSSL_TAR} \
# extract, configure, build \
    && tar -xf ${OPENSSL_TAR} \
    && cd openssl-${OPENSSL_VERSION} \
    && ./Configure enable-fips --libdir=lib --prefix=/usr \
# install the FIPS module \
    && make -j $(nproc) install_fips \
# clean up build deps and work area \
    && apk del build-deps \
    && cd / && rm -fr /work ~/.gnupg

# install openssl and then generate the fipsmodule.cnf configuration file from
# the built FIPS module. Modify the distribution openssl.cnf file to activate
# and use the built FIPS module.
RUN apk --no-cache --update add openssl \
# this eval sets OPENSSLDIR and MODULESDIR \
    && eval $(openssl version -d -m | sed 's/: /=/') \
    && FIPSCNF=${OPENSSLDIR}/fipsmodule.cnf \
    && OPENCNF=${OPENSSLDIR}/openssl.cnf \
    && openssl fipsinstall \
        -module ${MODULESDIR}/fips.so \
        -pedantic -out ${FIPSCNF} \
# edit the shipped openssl.cnf to enable the FIPS module \
    && sed -i \
        -e "s|# .include fipsmodule.cnf|.include ${FIPSCNF}|" \
        -e "s|# fips = fips_sect|fips = fips_sect\nbase = base_sect|" \
        -e "/providers = provider_sect/a alg_section = algorithm_sect" \
        ${OPENCNF} \
# add the [base_sect] and [algorithm_sect] sections to the end \
    && echo '' >> ${OPENCNF} \
    && echo '[base_sect]' >> ${OPENCNF} \
    && echo 'activate = 1' >> ${OPENCNF} \
    && echo '' >> ${OPENCNF} \
    && echo '[algorithm_sect]' >> ${OPENCNF} \
    && echo 'default_properties = fips=yes' >> ${OPENCNF}
# Run some sanity checks on the build, configuration \
# this first test is expected to fail because FIPS doesn't allow MD5 \
RUN  ! openssl md5 ${OPENCNF} \
    && openssl sha1 ${OPENCNF} \
    && openssl list -providers
