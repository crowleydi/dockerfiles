FROM docker.io/kanstin/alpine-fips:latest

ARG TOMCAT_VERSION=9.0.104
ENV INSTALL_BASE=/usr/local
ENV CATALINA_HOME=${INSTALL_BASE}/tomcat
ENV PATH=${CATALINA_HOME}/bin:$PATH

RUN apk --no-cache -U add \
    tomcat-native \
    openjdk17-jre-headless \
    ca-certificates \
# add GPG to verify signature \
    gpg gpg-agent \
# download known tomcat signer keys \
  && gpg --recv-keys \
    6FB21E8933C60243 \
    10C01C5A2F6059E7 \
    68248959359E722B \
  && TOMCAT_TAR=apache-tomcat-${TOMCAT_VERSION}.tar.gz \
  && URL=https://archive.apache.org/dist/tomcat/tomcat-9/v${TOMCAT_VERSION}/bin \
# download the tar, sha, and signature \
  && wget ${URL}/${TOMCAT_TAR} \
  && wget ${URL}/${TOMCAT_TAR}.sha512 \
  && wget ${URL}/${TOMCAT_TAR}.asc \
# verify the tar file 2x \
  && sha512sum -c ${TOMCAT_TAR}.sha512 \
  && gpg --verify ${TOMCAT_TAR}.asc ${TOMCAT_TAR} \
# extract and move it to the install base directory \
  && tar xf ${TOMCAT_TAR} \
  && mv apache-tomcat-${TOMCAT_VERSION} ${INSTALL_BASE} \
  && rm -f ${TOMCAT_TAR} ${TOMCAT_TAR}.* \
# link the installed tomcat directory to be CATALINA_HOME \
  && ln -s \
    ${INSTALL_BASE}/apache-tomcat-${TOMCAT_VERSION} \
    ${CATALINA_HOME} \
# don't use the shipped webapps\
  && mv \
    ${CATALINA_HOME}/webapps \
    ${CATALINA_HOME}/webapps.dist \
  && mkdir ${CATALINA_HOME}/webapps \
# cleanup GPG \
  && apk del gpg gpg-agent \
  && rm -fr ~/.gnupg/ \
# show Tomcat version to make sure things are installed and running \
  && openssl version \
  && catalina.sh version

WORKDIR $CATALINA_HOME
EXPOSE 8080

CMD ["catalina.sh", "run"]
