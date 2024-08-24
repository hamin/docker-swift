FROM redhat/ubi9

RUN groupadd -g 42 build-user && \
    useradd -m -r -u 42 -g build-user build-user

RUN yum install -y  \
  git               \
  gcc-c++           \
  cmake             \
  diffutils         \
  make              \
  libcurl-devel     \
  libedit-devel     \
  libuuid-devel     \
  libxml2-devel     \
  ncurses-devel     \
  python3-devel     \
  rsync             \
  sqlite-devel      \
  unzip             \
  zip

ARG SWIFT_PLATFORM=ubi9
ARG SWIFT_VERSION=5.9.2
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
    ARCH_NAME="$(rpm --eval '%{_arch}')"; \
    url=; \
    case "${ARCH_NAME##*-}" in \
        'x86_64') \
            OS_ARCH_SUFFIX=''; \
            ;; \
        'aarch64') \
            OS_ARCH_SUFFIX='-aarch64'; \
            ;; \
        *) echo >&2 "error: unsupported architecture: '$ARCH_NAME'"; exit 1 ;; \
    esac; \
    SWIFT_WEBDIR="$SWIFT_WEBROOT/$SWIFT_BRANCH/$(echo $SWIFT_PLATFORM | tr -d .)$OS_ARCH_SUFFIX" \
    && SWIFT_BIN_URL="$SWIFT_WEBDIR/$SWIFT_TAG/$SWIFT_TAG-$SWIFT_PLATFORM$OS_ARCH_SUFFIX.tar.gz" \
    && SWIFT_SIG_URL="$SWIFT_BIN_URL.sig" \
    && echo $SWIFT_BIN_URL \
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
