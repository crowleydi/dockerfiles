FROM docker.io/kanstin/alpine-fips:3.23-3.0.9

ARG TOMCAT_VERSION=9.0.115
ARG TCNATIVE_VERSION=2.0.13

ENV INSTALL_BASE=/usr/local
ENV CATALINA_HOME=${INSTALL_BASE}/tomcat
ENV PATH=${CATALINA_HOME}/bin:$PATH

RUN apk upgrade \
  && apk --no-cache add \
# going to remove these \
    build-base gpg gpg-agent openjdk21-jdk apr-dev openssl-dev \
# keep these \
    openjdk21-jre-headless apr ca-certificates \
# download known tomcat signer keys \
  && mkdir /work \
  && cd /work \
  && gpg --recv-keys \
# fingerprints for apache-tomcat signing keys \
      'DCFD 35E0 BF8C A734 4752 DE8B 6FB2 1E89 33C6 0243' \
      'A9C5 DF4D 22E9 9998 D987 5A51 10C0 1C5A 2F60 59E7' \
      '48F8 E69F 6390 C9F2 5CFE DCD2 6824 8959 359E 722B' \
# additional fingerprints for tomcat-native signing keys \
      '3E6A C004 854F 3A7F 0356 6B59 2FF0 6894 E55B 0D0E' \
      '3A6F 081D DFD3 DE93 02C3 1329 0F45 0A26 6210 BFC0' \
      '4DBE BB77 94DD A20A 01F4 482A 9038 4D9A 35C7 E942' \
      '7E1B 5FEC 62E5 56E1 C098 7E78 D21C D223 EE8E 0DD5' \
      '9BA4 4C26 2138 5CB9 66EB A586 F72C 284D 731F ABEE' \
  && APACHE_URL="https://archive.apache.org/dist/tomcat" \
  && TOMCAT_DIR="apache-tomcat-${TOMCAT_VERSION}" \
  && TOMCAT_TAR="${TOMCAT_DIR}.tar.gz" \
  && TOMCAT_URL="${APACHE_URL}/tomcat-9/v${TOMCAT_VERSION}/bin" \
# download the tar, sha, and signature \
  && wget "${TOMCAT_URL}/${TOMCAT_TAR}" \
  && wget "${TOMCAT_URL}/${TOMCAT_TAR}.sha512" \
  && wget "${TOMCAT_URL}/${TOMCAT_TAR}.asc" \
# verify the tar file 2x \
  && sha512sum -c "${TOMCAT_TAR}.sha512" \
  && gpg --verify "${TOMCAT_TAR}.asc" "${TOMCAT_TAR}" \
# extract and move it to the install base directory \
  && tar xf "${TOMCAT_TAR}" \
  && mv "${TOMCAT_DIR}" "${INSTALL_BASE}" \
# link the installed tomcat directory to be CATALINA_HOME \
  && ln -s "${INSTALL_BASE}/${TOMCAT_DIR}" "${CATALINA_HOME}" \
# don't use the shipped webapps\
  && mv "${CATALINA_HOME}/webapps" "${CATALINA_HOME}/webapps.dist" \
  && mkdir "${CATALINA_HOME}/webapps" \
# download tomcat-native \
  && TCNATIVE_DIR="tomcat-native-${TCNATIVE_VERSION}-src" \
  && TCNATIVE_TAR="${TCNATIVE_DIR}.tar.gz" \
  && TCNATIVE_URL="${APACHE_URL}/tomcat-connectors/native/${TCNATIVE_VERSION}/source" \
  && wget "${TCNATIVE_URL}/${TCNATIVE_TAR}" \
  && wget "${TCNATIVE_URL}/${TCNATIVE_TAR}.sha512" \
  && wget "${TCNATIVE_URL}/${TCNATIVE_TAR}.asc" \
# verify the tar file 2x \
  && sha512sum -c "${TCNATIVE_TAR}.sha512" \
  && gpg --verify "${TCNATIVE_TAR}.asc" "${TCNATIVE_TAR}" \
# compile tomcat-native \
  && tar xf "${TCNATIVE_TAR}" \
  && cd "${TCNATIVE_DIR}/native" \
  && ./configure \
      --with-apr=/usr/bin/apr-1-config \
      --with-java-home=/usr/lib/jvm/java-21-openjdk \
  && make -j $(nproc) \
  && mv .libs/libtcnative*.so* /usr/lib \
# test tomcat native is working \
  && configtest=$(catalina.sh configtest 2>&1) \
  && echo "$configtest" | grep -q "Loaded Apache Tomcat Native" \
  && echo "$configtest" | grep -q "Using OpenSSL with the FIPS provider" \
  && echo "$configtest" | grep -q "OpenSSL successfully initialized" \
# cleanup GPG, files, and progs we don't need\
  && cd / \
  && rm -fr /work ~/.gnupg/ \
  && apk del \
    build-base gpg gpg-agent openjdk21-jdk apr-dev openssl-dev \
# setup tomcat user \
  && adduser -D -H tomcat tomcat \
  && chown -R tomcat:tomcat \
      "${INSTALL_BASE}/${TOMCAT_DIR}/logs" \
      "${INSTALL_BASE}/${TOMCAT_DIR}/temp" \
      "${INSTALL_BASE}/${TOMCAT_DIR}/work" \
# show Tomcat and OpenSSL version info to make sure \
# things are installed and running \
  && openssl version \
  && catalina.sh version

WORKDIR ${INSTALL_BASE}/apache-tomcat-${TOMCAT_VERSION}
EXPOSE 8080
USER tomcat
CMD ["catalina.sh", "run"]
