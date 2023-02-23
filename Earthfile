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
  # autoconf does not recognize aarch64 -- use arm instead
  # https://github.com/tpoechtrager/cctools-port/issues/6
  IF [ $architecture = "aarch64" ]
    ARG triple=arm-apple-darwin$kernel_version
  ELSE
    ARG triple=$architecture-apple-darwin$kernel_version
  END
  FROM +deps
  RUN apt install -y clang llvm-dev uuid-dev git rename
  ARG cctools_version=973.0.1
  ARG linker_verison=609
  GIT CLONE --branch $cctools_version-ld64-$linker_verison https://github.com/tpoechtrager/cctools-port .
  # TODO: remove the need for this patch... somehow
  # https://github.com/iains/gcc-darwin-arm64/issues/102
  IF [ $architecture = "aarch64" ]
    COPY cctools.patch .
    RUN git apply cctools.patch
  END
  COPY +xar/ /xar
  COPY +libtapi/ /libtapi
  WORKDIR cctools
  RUN ./configure \
    --prefix=/cctools \
    --with-libtapi=/libtapi \
    --with-libxar=/libxar \
    --disable-clang-as \
    --target=$triple
  # now that we've tricked autoconf by pretending to build for arm, let's _actually_ build for arm64
  # https://github.com/tpoechtrager/cctools-port/issues/6
  RUN find . -name Makefile -print0 | xargs -0 sed -i "s/arm-apple-darwin$kernel_version/arm64-apple-darwin$kernel_version/g"
  RUN make -j
  RUN make install
  # output arm64 artifacts as aarch64 so that the target triple is consistent with what clang/gcc will expect
  RUN rename "s/arm64-apple-darwin$kernel_version/aarch64-apple-darwin$kernel_version/" /cctools/bin/*
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
  ARG triple=$architecture-apple-darwin$kernel_version
  FROM +deps
  RUN apt install -y gcc g++ zlib1g-dev libmpc-dev libmpfr-dev libgmp-dev flex file

  # TODO: this shouldn't be needed
  RUN apt-get install -y --force-yes clang llvm-dev libxml2-dev uuid-dev libssl-dev bash patch make tar xz-utils bzip2 gzip sed cpio libbz2-dev zlib1g-dev

  IF [ $architecture = "aarch64" ]
    GIT CLONE --branch master-wip-apple-si https://github.com/iains/gcc-darwin-arm64 gcc
  ELSE IF [ $architecture = "x86_64" ]
    GIT CLONE --branch releases/gcc-12 https://github.com/gcc-mirror/gcc gcc
  ELSE
    RUN false
  END

  COPY (+cctools/ --kernel_version=$kernel_version) /cctools
  ENV PATH=$PATH:/cctools/bin/
  
  COPY (+sdk/ --version=$sdk_version) /sdk

  # TODO: I think we can remove these
  COPY +xar/ /sdk/usr
  COPY +libtapi/ /sdk/usr

  COPY +xar/lib /usr/local/lib
  COPY +libtapi/lib /usr/local/lib
  RUN ldconfig

  # GCC requires that you build in a directory that is not a subdirectory of the source code
  # https://gcc.gnu.org/install/configure.html
  WORKDIR build
  RUN ../gcc/configure \
    --target=$triple \
    --with-sysroot=/sdk \
    --disable-nls \
    --enable-languages=c,c++ \ # ,fortran,objc,obj-c++ \ # TODO: enable rust?
    --without-headers \
    --enable-lto \
    --enable-checking=release \
    --disable-libstdcxx-pch \ # TODO: maybe enable this
    --prefix=/gcc \
    --with-system-zlib \
    --disable-multilib
  RUN make -j4
  RUN make install
  SAVE ARTIFACT /gcc/*

image:
  ARG architectures=aarch64 #x86_64
  ARG sdk_version=13.0
  ARG kernel_version=22
  FROM ubuntu:jammy
  RUN apt update
  RUN apt install -y clang
  FOR architecture IN $architectures
    COPY (+cctools/ --architecture=$architecture --sdk_version=$sdk_version --kernel_version=$kernel_version) /usr/local
    COPY (+gcc/ --architecture=$architecture --sdk_version=$sdk_version --kernel_version=$kernel_version) /usr/local
  END
  COPY (+sdk/ --version=$sdk_version) /sdk
  SAVE IMAGE macos-cross-compiler

test:
  ARG architectures=aarch64 #x86_64
  ARG sdk_version=13.0
  ARG kernel_version=22
  ARG triple=$architecture-apple-darwin$kernel_version
  FROM +image
  COPY samples samples
  FOR architecture IN architectures
    RUN $triple-gcc samples/hello.c -o hello
    RUN $triple-g++ samples/hello.cpp -o hello
    RUN $triple-gfortran samples/hello.f90 -o hello
    RUN clang --target=$triple samples/hello.c -o hello
    RUN clang++ --target=$triple samples/hello.cpp -o hello
  END
