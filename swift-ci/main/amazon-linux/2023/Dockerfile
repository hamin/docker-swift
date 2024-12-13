FROM amazonlinux:2023

RUN yum install shadow-utils -y

ARG BUILD_USER_ID=998

RUN groupadd -g ${BUILD_USER_ID} build-user && \
    useradd -m -r -u ${BUILD_USER_ID} -g build-user build-user

RUN yum -y group install "development tools"
RUN yum -y install \
  cmake \
  ninja-build \
  curl-devel \
  gcc-c++ \
  clang \
  git \
  libbsd-devel \
  libedit-devel \
  libicu-devel \
  libuuid-devel \
  libxml2-devel \
  ncurses-devel \
  pkgconfig \
  procps-ng \
  python3 \
  python3-devel \
  python3-distro \
  python3-setuptools \
  rsync \
  sqlite-devel \
  swig \
  tzdata \
  unzip \
  zip \
  lld \
  diffutils


# TODO: Still need to install bits for lldb

USER build-user
WORKDIR /home/build-user
