FROM docker.io/kanstin/alpine-fips:latest

ARG TOMCAT_VERSION=9.0.104
ENV TOMCAT_TAR=apache-tomcat-${TOMCAT_VERSION}.tar.gz
ENV URL=https://archive.apache.org/dist/tomcat/tomcat-9/v${TOMCAT_VERSION}/bin
ENV INSTALL_BASE=/usr/local
ENV CATALINA_HOME=${INSTALL_BASE}/tomcat
ENV PATH=$CATALINA_HOME/bin:$PATH

RUN wget ${URL}/${TOMCAT_TAR} \
  && wget ${URL}/${TOMCAT_TAR}.sha512 \
  && sha512sum -c ${TOMCAT_TAR}.sha512 \
  && tar xf ${TOMCAT_TAR} \
  && mv apache-tomcat-${TOMCAT_VERSION} ${INSTALL_BASE} \
  && rm -f ${TOMCAT_TAR} ${TOMCAT_TAR}.sha512 \
  && apk --no-cache -U add tomcat-native openjdk17-jre-headless ca-certificates \
	&& ln -s ${INSTALL_BASE}/apache-tomcat-${TOMCAT_VERSION} ${CATALINA_HOME} \
	&& mv ${CATALINA_HOME}/webapps ${CATALINA_HOME}/webapps.dist \
	&& mkdir ${CATALINA_HOME}/webapps \
	&& catalina.sh version

WORKDIR $CATALINA_HOME
EXPOSE 8080

CMD ["catalina.sh", "run"]
