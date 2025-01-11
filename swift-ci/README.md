# Continuous Integration Docker images

Swift.org uses Docker based virtual build environment to build & qualify Swift toolchains on Linux.

The Continuous Integration system uses the Dockerfiles in this directory to define the virtual build environment, then runs the build and qualification steps inside a docker container based on the image.

## Directory Structure

The Dockerfiles used for Continuous Integration are layed out under the top level `swift-ci` directory. Under that, we have a directory for each of the target branches, e.g. Continuous Integration for Swift's `main` branch uses the `swift-ci/master` Dockerfiles.

There is also a specific directory (`swift-docc-render`) for the Dockerfile used to build Swift-DocC-Render. Swift-DocC-Render builds separately from the rest of the projects in the Swift toolchain and ships a pre-built copy for use in the toolchain in the Swift-DocC-Render-Artifact repository.

## Continuous Integration

This system is designed to support many distributions. Once a working Dockerfile is added to this repository, we set up Continuous Integration jobs to produce toolchains for the distribution and publish them on Swift.org.

## Local development & testing

First build & tag the Dockerfile:

```bash
docker build -f <dockerfile path> . -t <some tag>
```

Next, run the Swift build using that Docker image:

```bash
docker run \
  --cap-add=SYS_PTRACE \
  --security-opt seccomp=unconfined \
  -v <swift source location>:/source \
  -v <some docker volume name>:/home/build-user \
  -w /home/build-user/ \
  <docker image tag from above> \
  /bin/bash -lc "cp -r /source/* /home/build-user/; ./swift/utils/build-script --preset buildbot_linux install_destdir=/home/build-user/swift-install installable_package=/home/build-user/swift-DEVELOPMENT-SNAPSHOT-$(date +'%F')-a.tar.gz"
```

## Quick Start for Windows Development

The Windows Docker image will setup an enviornment with Python, Visual Studio
Build Tools, and Git.  It is setup to assume that the sources will be available
in `S:\SourceCache`.

### Building and Tagging the image

Windows docker images must match the kernel version in the container and the
host.  You can identify the correct version by runing the `winver` command.  The
"OS Build" identifies the version suffix to apply to "10.0".

```powershell
cd master\windows\10.0.19044.1706
docker image build --compress -t swift:swiftci .
```

### Running the image

```powershell
docker run --rm -it -v %UserProfile%\data:S: swift:swiftci
```

### Building the Toolchain

While we can build the toolchain in the containerized environment, the sources
are expected to reside on the host and is passed into the docker container as a
volume.  The rest of automation expects that the Sources reside under a
directory with the name `SourceCache`.

#### Clone the Sources

```cmd
md %UserProfile%\data\SourceCache
cd %UserProfile%\data\SourceCache

git clone -b stable/20220426 -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/apple/llvm-project
git clone -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/apple/swift
git clone -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/apple/swift-cmark cmark
git clone -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/apple/swift-experimental-string-processing
git clone -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/apple/swift-corelibs-libdispatch
git clone -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/apple/swift-corelibs-foundation
git clone -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/apple/swift-corelibs-xctest
git clone -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/apple/swift-argument-parser
git clone -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/apple/swift-crypto
git clone -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/apple/swift-driver
git clone -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/apple/swift-llbuild llbuild
git clone -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/apple/swift-package-manager
git clone -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/apple/swift-system
git clone -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/apple/swift-tools-support-core
git clone -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/apple/swift-installer-scripts
git clone -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/apple/indexstore-db
git clone -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/apple/sourcekit-lsp
git clone -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/jpsim/Yams
git clone -t curl-7_77_0 -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/curl/curl
git clone -t v2.9.12 -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/gnome/libxml2
git clone -t v1.2.11 -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/madler/zlib
git clone -b maint/maint-69 -c core.autocrlf=input -c core.symlink=true -c core.useBuiltinFSMonitor=false https://github.com/unicode-org/icu
```

#### Run Docker

```cmd
docker run --rm -it -v %UserProfile%\data:S: swift:swiftci
```

#### Build the Toolchain

This will build a full Swift toolchain distribution (llvm, clang, lld, lldb,
swift, swift-package-manger, SourceKit-LSP) and the Windows SDK (x86, x64,
ARM64).

```cmd
S:
S:\SourceCache\swift\utils\build.cmd
```

#### Running Swift Tests

The toolchain tests require some modifications to the path to find some of the
dependencies.  The following will run the Swift test suite within the docker
container:

```cmd
path S:\b\1\bin;S:\b\1\tools\swift\libdispatch-windows-x86_64-prefix\bin;%Path%;%ProgramFiles%\Git\usr\bin
ninja -C S:\b\1 check-swift
```

#### Using the Toolchain

> **NOTE**: Running the test suite and using the toolchain near the production mode are mututally incompatible (due to the path changes).

The build will generate a toolchain image that is roughly similar to the
installed version.  The following can be run inside the docker container to use
the toolchain:

```cmd
set SDKROOT=S:\Library\Developer\Platforms\Windows.platform\Developer\SDKs\Windows.sdk
path S:\Library\Developer\Platforms\Windows.platform\Developer\SDKs\Windows.sdk\usr\bin\x64;S:\Library\Developer\Toolchains\unknown-Asserts-development.xctoolchain\usr\bin;%Path%
```

Because the toolchain is built in the volume which is backed by the host, the
toolchain can be used on the host (assuming the dependencies such as Visual
Studio is installed and the module modules deployed).  The adjusted paths below
should enable that:

```cmd
set SDKROOT=%UserProfile%\data\Library\Developer\Platforms\Windows.platform\Developer\SDKs\Windows.sdk
path %UserProfile%\data\Library\Developer\Platforms\Windows.platform\Developer\SDKs\Windows.sdk\usr\bin\x64;%UserProfile%\data\Library\Developer\Toolchains\unknown-Asserts-development.xctoolchain\usr\bin;%Path%
```

## Contributions

Contributions via pull requests are welcome and encouraged :)

Focus on mainstream distributions such as Debian, Ubuntu, Fedora, CentOS, RedHat, etc.

Note that the build must run as non-root user.
