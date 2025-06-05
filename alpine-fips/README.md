# Alpine Linux with FIPS OpenSSL module

## Dockerfile

This Dockerfile uses Alpine Linux as a base image. It installs necessary build
tools and then downloads the latest FIPS source for OpenSSL (version 3.1.2).
The downloaded source is chacked agaonst the sha256 checksum and then the GPG
signature is verified. The FIPS module is built from source. The build process
is based on the instructions give in the [OpenSSL FIPS
support guide](https://github.com/openssl/openssl/blob/master/README-FIPS.md).

Once the FIPS provider is installed, all source and build tools are removed
from the image. The latest `openssl` package is installed and the OpenSSL
configuration file is updated to activate the built FIPS provider module.
