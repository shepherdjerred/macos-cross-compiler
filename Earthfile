VERSION 0.7

FROM ubuntu:jammy
WORKDIR /workspace

deps:
  RUN apt update -y
  RUN apt install -y build-essential cmake

xar:
  FROM +deps
  RUN apt install -y libxml2-dev libssl-dev zlib1g-dev
  GIT CLONE https://github.com/tpoechtrager/xar .
  WORKDIR xar
  RUN ./configure --prefix=/xar
  RUN make -j
  RUN make install
  SAVE ARTIFACT /xar/*

libtapi:
  FROM +deps
  RUN apt install -y python3
  ARG version=1100.0.11
  GIT CLONE --branch $version https://github.com/tpoechtrager/apple-libtapi .
  ENV INSTALLPREFIX=/libtapi
  RUN ./build.sh
  RUN ./install.sh
  SAVE ARTIFACT /libtapi/*

cctools:
  ARG --required architecture
  ARG --required kernel_version
  FROM +deps
  RUN apt install -y clang llvm-dev uuid-dev
  ARG cctools_version=973.0.1
  ARG linker_verison=609
  GIT CLONE --branch $cctools_version-ld64-$linker_verison https://github.com/tpoechtrager/cctools-port .
  COPY +xar/ /xar
  COPY +libtapi/ /libtapi
  WORKDIR cctools
  RUN ./configure \
    --prefix=/cctools \
    --with-libtapi=/libtapi \
    --with-libxar=/libxar \
    --target=$architecture-apple-darwin$kernel_version
  RUN make -j
  RUN make install
  SAVE ARTIFACT /cctools/*

sdk:
  ARG --required version
  FROM +deps
  COPY sdks/MacOSX$version.sdk.tar.xz .
  RUN tar -xf MacOSX$version.sdk.tar.xz
  SAVE ARTIFACT MacOSX$version.sdk/*

gcc:
  ARG --required architecture
  ARG --required sdk_version
  ARG --required kernel_version
  FROM +deps
  RUN apt install -y gcc g++ zlib1g-dev libmpc-dev libmpfr-dev libgmp-dev flex file
  IF [ $architecture = "aarch64" ]
    GIT CLONE --branch master-wip-apple-si https://github.com/iains/gcc-darwin-arm64 .
  ELSE IF [ $architecture = "x86_64" ]
    GIT CLONE --branch releases/gcc-12 https://github.com/gcc-mirror/gcc .
  ELSE
    RUN false
  END
  COPY (+cctools/ --kernel_version=$kernel_version) /cctools
  COPY (+sdk/ --version=$sdk_version) /sdk
  ENV PATH=$PATH:/cctools/bin/
  RUN ./configure \
    --target=$architecture-apple-darwin$kernel_version \
    --with-sysroot=/sdk \
    --disable-nls \
    --enable-languages=c,c++,objc,obj-c++,fortran \
    --without-headers \
    --enable-lto \
    --enable-checking=release \
    --disable-libstdcxx-pch \
    --prefix=/gcc \
    --with-system-zlib \
    --disable-multilib
  RUN make -j2
  RUN make install
  SAVE ARTIFACT /gcc/*

image:
  ARG architecture=aarch64
  ARG sdk_version=13.0
  ARG kernel_version=22
  FROM ubuntu:jammy
  RUN apt update
  COPY +gcc/* /usr/local/bin
  COPY +cctools/* /usr/local/bin
