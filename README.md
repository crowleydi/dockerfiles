# dockerfiles
Miscellaneous dockerfiles for building useful containers.

## alpine-fips

This is Alpine Linux v3.21 plus OpenSSL with the FIPS module.

[![Build alpine-fips Docker image](https://github.com/crowleydi/dockerfiles/actions/workflows/build-alpine-fips.yml/badge.svg)](https://github.com/crowleydi/dockerfiles/actions/workflows/build-alpine-fips.yml)

## alpine-tomcat

Tomcat 9.0.x built using alpine-fips as the base image. Tomcat is installed such that
one can easily use this image instead of the official Tomcat image.

[![Build alpine-tomcat Docker image](https://github.com/crowleydi/dockerfiles/actions/workflows/build-alpine-tomcat.yml/badge.svg)](https://github.com/crowleydi/dockerfiles/actions/workflows/build-alpine-tomcat.yml)

## alpine-apache

This is Apache httpd 2.4.x plus OpenSSL with the FIPS module.

[![Build alpine-tomcat Docker image](https://github.com/crowleydi/dockerfiles/actions/workflows/build-alpine-apache.yml/badge.svg)](https://github.com/crowleydi/dockerfiles/actions/workflows/build-alpine-apache.yml)

