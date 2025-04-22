# dockerfiles
Miscellaneous dockerfiles for building useful containers.

## alpine-fips

This is Alpine Linux v3.21 plus OpenSSL with the FIPS module.

[![Build alpine-fips Docker image](https://github.com/crowleydi/dockerfiles/actions/workflows/docker-image.yml/badge.svg)](https://github.com/crowleydi/dockerfiles/actions/workflows/docker-image.yml)

## alpine-tomcat

Tomcat 9.0.x built using alpine-fips as the base image. Tomcat is installed such that
one can easily use this image instead of the official Tomcat image.
