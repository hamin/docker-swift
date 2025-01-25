FROM redhat/ubi8

RUN groupadd -g 42 build-user && \
    useradd -m -r -u 42 -g build-user build-user

RUN yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

# Update and install needed build packages
RUN yum -y install \
  autoconf              \
  autoconf              \
  clang                 \
  diffutils             \
  git                   \
  glibc-static          \
  libbsd-devel          \
  libcurl-devel         \
  libedit-devel         \
  libicu-devel          \
  libstdc++-static      \
  libtool               \
  libuuid-devel         \
  libxml2-devel         \
  make                  \
  ncurses-devel         \
  pcre-devel            \
  platform-python-devel \
  python2               \
  python2-devel         \
  python2-six           \
  python3               \
  python3-six           \
  rsync                 \
  sqlite-devel          \
  swig                  \
  tar                   \
  which

RUN alternatives --install /usr/bin/pip pip /usr/bin/pip2 1
RUN alternatives --set python /usr/bin/python2

USER build-user

WORKDIR /home/build-user
