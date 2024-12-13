FROM centos:7

RUN yum install shadow-utils.x86_64 -y

ARG BUILD_USER_ID=998

RUN groupadd -g ${BUILD_USER_ID} build-user && \
    useradd -m -r -u ${BUILD_USER_ID} -g build-user build-user

RUN yum install -y epel-release centos-release-scl

RUN yum install --enablerepo=centosplus  -y \
    autoconf            \
    cmake               \
    devtoolset-8        \
    glibc-static        \
    libatomic           \
    libcurl-devel       \
    libedit-devel       \
    libstdc++-static    \
    libtool             \
    libuuid-devel       \
    libxml2-devel       \
    make                \
    ncurses-devel       \
    ninja-build         \
    openssl-devel       \
    pexpect             \
    python-devel        \
    python3-devel       \
    python-pygments     \
    python-six          \
    python36-pexpect    \
    python36-six        \
    PyYAML              \
    rsync               \
    sclo-git25-git      \
    sqlite-devel        \
    swig3               \
    which               \
    unzip               \
    zip                 \
    zlib-devel

RUN echo -e ". /opt/rh/sclo-git25/enable\n. /opt/rh/devtoolset-8/enable\n" >> /home/build-user/.bashrc

RUN sed -i -e 's/\*__block/\*__libc_block/g' /usr/include/unistd.h

ARG SWIFT_PLATFORM=centos7
ARG SWIFT_VERSION=5.8.1
ARG SWIFT_BRANCH=swift-${SWIFT_VERSION}-release
ARG SWIFT_TAG=swift-${SWIFT_VERSION}-RELEASE
ARG SWIFT_WEBROOT=https://download.swift.org
ARG SWIFT_PREFIX=/opt/swift/${SWIFT_VERSION}

ENV SWIFT_PLATFORM=$SWIFT_PLATFORM \
    SWIFT_VERSION=$SWIFT_VERSION \
    SWIFT_BRANCH=$SWIFT_BRANCH \
    SWIFT_TAG=$SWIFT_TAG \
    SWIFT_WEBROOT=$SWIFT_WEBROOT \
    SWIFT_PREFIX=$SWIFT_PREFIX

RUN set -e; \
    SWIFT_WEBDIR="$SWIFT_WEBROOT/$SWIFT_BRANCH/$(echo $SWIFT_PLATFORM | tr -d .)" \
    && SWIFT_BIN_URL="$SWIFT_WEBDIR/$SWIFT_TAG/$SWIFT_TAG-$SWIFT_PLATFORM.tar.gz" \
    && SWIFT_SIG_URL="$SWIFT_BIN_URL.sig" \
    # - Download the GPG keys, Swift toolchain, and toolchain signature, and verify.
    && export GNUPGHOME="$(mktemp -d)" \
    && curl -fsSL "$SWIFT_BIN_URL" -o swift.tar.gz "$SWIFT_SIG_URL" -o swift.tar.gz.sig \
    && curl -fSsL https://swift.org/keys/all-keys.asc | gpg --import -  \
    && gpg --batch --verify swift.tar.gz.sig swift.tar.gz \
    # - Unpack the toolchain, set libs permissions, and clean up.
    && mkdir -p $SWIFT_PREFIX \
    && tar -xzf swift.tar.gz --directory $SWIFT_PREFIX --strip-components=1 \
    && chmod -R o+r $SWIFT_PREFIX/usr/lib/swift \
    && rm -rf "$GNUPGHOME" swift.tar.gz.sig swift.tar.gz

ENV PATH="${SWIFT_PREFIX}/usr/bin:${PATH}"

USER build-user

WORKDIR /home/build-user

ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
