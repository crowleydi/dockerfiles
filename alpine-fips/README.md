# Alpine Linux with FIPS OpenSSL module

## Dockerfile

This Dockerfile uses Alpine Linux as a base image. It installs necessary build
tools and then downloads the latest FIPS validated source for openssl (version
3.0.9). The downloaded source is validated with the sha256 checksum. The FIPS
module is built from source and the build process is based on the [OpenSSL FIPS
support guide](https://github.com/openssl/openssl/blob/master/README-FIPS.md).

Once the FIPS provider is installed, all source and build tools are removed
from the image. The latest `openssl` package is installed and the configuration
file updated to use the built FIPS provider.
