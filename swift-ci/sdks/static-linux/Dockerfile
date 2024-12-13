# ===----------------------------------------------------------------------===
#
#  Swift Static SDK for Linux: Docker-based build
#
#  This source file is part of the Swift.org open source project
#
#  Copyright (c) 2024 Apple Inc. and the Swift project authors
#  Licensed under Apache License v2.0 with Runtime Library Exception
#
#  See https://swift.org/LICENSE.txt for license information
#  See https://swift.org/CONTRIBUTORS.txt for the list of Swift project authors
#
# ===----------------------------------------------------------------------===

FROM ubuntu:22.04

# Versions to fetch

ARG SWIFT_VERSION=scheme:release/6.0
ARG MUSL_VERSION=1.2.5
ARG LIBXML2_VERSION=2.12.7
ARG CURL_VERSION=8.7.1
ARG BORINGSSL_VERSION=fips-20220613
ARG ICU_VERSION=maint/maint-69
ARG ZLIB_VERSION=1.3.1

# Architecture to build on (empty means x86-64)
ARG OS_ARCH_SUFFIX=

# ............................................................................

# Install development tools
RUN apt-get -q update \
    && DEBIAN_FRONTEND=noninteractive apt-get -q install -y \
    build-essential \
    cmake \
    ninja-build \
    python3 \
    golang \
    git \
    gnupg2 \
    libsqlite3-dev \
    libcurl4-openssl-dev  \
    libedit-dev           \
    libicu-dev            \
    libncurses5-dev       \
    libpython3-dev        \
    libsqlite3-dev        \
    libxml2-dev           \
    uuid-dev \
    uuid-runtime \
    tzdata \
    curl \
    && rm -rf /var/lib/apt-lists/*

# Install Swift
ARG SWIFT_SIGNING_KEY=E813C892820A6FA13755B268F167DF1ACF9CE069
ARG SWIFT_PLATFORM=ubuntu
ARG OS_MAJOR_VER=22
ARG OS_MINOR_VER=04
ARG SWIFT_WEBROOT=https://download.swift.org/swift-6.0-branch

ENV SWIFT_SIGNING_KEY=$SWIFT_SIGNING_KEY \
    SWIFT_PLATFORM=$SWIFT_PLATFORM \
    OS_MAJOR_VER=$OS_MAJOR_VER \
    OS_MINOR_VER=$OS_MINOR_VER \
    OS_VER=$SWIFT_PLATFORM$OS_MAJOR_VER.$OS_MINOR_VER \
    SWIFT_WEBROOT="$SWIFT_WEBROOT/$SWIFT_PLATFORM$OS_MAJOR_VER$OS_MINOR_VER$OS_ARCH_SUFFIX"

COPY scripts/install-swift.sh /scripts/install-swift.sh
RUN chmod ugo+x /scripts/install-swift.sh

RUN /scripts/install-swift.sh

ENV PATH="/usr/local/swift/bin:${PATH}"

ENV SWIFT_VERSION=$SWIFT_VERSION \
    MUSL_VERSION=$MUSL_VERSION \
    LIBXML2_VERSION=$LIBXML2_VERSION \
    CURL_VERSION=$CURL_VERSION \
    BORINGSSL_VERSION=$BORINGSSL_VERSION \
    ICU_VERSION=$ICU_VERSION \
    ZLIB_VERSION=$ZLIB_VERSION

COPY scripts /scripts
RUN chmod ugo+x /scripts/*

COPY resources /resources

# Create a user
ARG BUILD_USER_ID=998

RUN groupadd -g ${BUILD_USER_ID} build-user && \
    useradd -m -r -u ${BUILD_USER_ID} -g build-user build-user

USER build-user

WORKDIR /home/build-user
