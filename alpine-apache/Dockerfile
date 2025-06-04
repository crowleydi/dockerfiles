# build arguments
ARG BASE_VERSION=2.4.63

# mount alpine-fips so we can copy over the FIPS module
FROM kanstin/alpine-fips:latest AS source

# base image
FROM httpd:${BASE_VERSION}-alpine

# copy the fips module from alpine-fips
COPY --from=source /usr/lib/ossl-modules/fips.so /usr/lib/ossl-modules

# add openssl
RUN apk --no-cache --update add openssl \
# this eval sets OPENSSLDIR and MODULESDIR \
    && eval $(openssl version -d -m | sed 's/: /=/') \
    && FIPSCNF=${OPENSSLDIR}/fipsmodule.cnf \
    && OPENCNF=${OPENSSLDIR}/openssl.cnf \
# create fipsmodule.cnf \
    && openssl fipsinstall \
        -module ${MODULESDIR}/fips.so \
        -pedantic -out ${FIPSCNF} \
# sed to edit the shipped openssl.cnf to enable the FIPS module \
    && sed -i \
        -e "s|# .include fipsmodule.cnf|.include ${FIPSCNF}|" \
        -e "s|# fips = fips_sect|fips = fips_sect\nbase = base_sect|" \
        -e "/providers = provider_sect/a alg_section = algorithm_sect" \
        ${OPENCNF} \
# append [base_sect] and [algorithn_sect] to the end \
    && echo '' >> ${OPENCNF} \
    && echo '[base_sect]' >> ${OPENCNF} \
    && echo 'activate = 1' >> ${OPENCNF} \
    && echo '' >> ${OPENCNF} \
    && echo '[algorithm_sect]' >> ${OPENCNF} \
    && echo 'default_properties = fips=yes' >> ${OPENCNF} \
# edit httpd.conf to enable https \
    && sed -i \
        -e 's/^#\(Include .*httpd-ssl.conf\)/\1/' \
        -e 's/^#\(LoadModule .*mod_ssl.so\)/\1/' \
        -e 's/^#\(LoadModule .*mod_socache_shmcb.so\)/\1/' \
        conf/httpd.conf
# run smoke-screen tests
RUN  ! openssl md5 ${OPENCNF} \
    && openssl sha1 ${OPENCNF} \
    && openssl list -providers
