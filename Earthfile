VERSION 0.7

FROM ubuntu:jammy
WORKDIR /workspace

deps:
  RUN apt update -y
  RUN apt install -y build-essential cmake clang git

xar:
  FROM +deps
  ARG --required target_sdk_version
  RUN apt install -y libxml2-dev libssl-dev zlib1g-dev
  GIT CLONE https://github.com/tpoechtrager/xar .
  WORKDIR xar
  ENV MACOSX_DEPLOYMENT_TARGET=$target_sdk_version
  RUN ./configure --prefix=/xar
  RUN make -j
  RUN make install
  SAVE ARTIFACT /xar/*

libtapi:
  FROM +deps
  RUN apt install -y python3
  ARG --required target_sdk_version
  ARG version=1100.0.11
  GIT CLONE --branch $version https://github.com/tpoechtrager/apple-libtapi .
  ENV MACOSX_DEPLOYMENT_TARGET=$target_sdk_version
  ENV INSTALLPREFIX=/libtapi
  RUN ./build.sh
  RUN ./install.sh
  SAVE ARTIFACT /libtapi/*

cctools:
  ARG --required architecture
  ARG --required kernel_version
  ARG --required target_sdk_version
  # autoconf does not recognize aarch64 -- use arm instead
  # https://github.com/tpoechtrager/cctools-port/issues/6
  IF [ $architecture = "aarch64" ]
    ARG triple=arm-apple-darwin$kernel_version
  ELSE
    ARG triple=$architecture-apple-darwin$kernel_version
  END
  FROM +deps
  RUN apt install -y llvm-dev uuid-dev rename
  ARG cctools_version=973.0.1
  ARG linker_verison=609
  GIT CLONE --branch $cctools_version-ld64-$linker_verison https://github.com/tpoechtrager/cctools-port .
  COPY (+xar/ --target_sdk_version=$target_sdk_version) /xar
  COPY (+libtapi/ --target_sdk_version=$target_sdk_version) /libtapi
  WORKDIR cctools
  ENV MACOSX_DEPLOYMENT_TARGET=$target_sdk_version
  RUN ./configure \
    --prefix=/cctools \
    --with-libtapi=/libtapi \
    --with-libxar=/libxar \
    --target=$triple
  # now that we've tricked autoconf by pretending to build for arm, let's _actually_ build for arm64
  # https://github.com/tpoechtrager/cctools-port/issues/6
  RUN find . -name Makefile -print0 | xargs -0 sed -i "s/arm-apple-darwin$kernel_version/arm64-apple-darwin$kernel_version/g"
  RUN make -j
  RUN make install
  # link aarch64 artifacts so that the target triple is consistent with what clang/gcc will expect
  IF [ $architecture = "aarch64" ]
    FOR file IN $(ls /cctools/bin/*)
      RUN /bin/bash -c "ln -s $file \${file/arm64/"aarch64"} "
    END
  END
  ENV PATH=$PATH:/cctools/bin
  SAVE ARTIFACT /cctools/*

wrapper:
  ARG --required sdk_version
  ARG --required kernel_version
  ARG --required target_sdk_version
  FROM +deps
  RUN apt install -y
  GIT CLONE https://github.com/tpoechtrager/osxcross .
  WORKDIR wrapper
  ENV VERSION=1.4
  ENV SDK_VERSION=$sdk_version
  ENV TARGET=darwin$kernel_version
  ENV LINKER_VERSION=609
  ENV X86_64H_SUPPORTED=0
  ENV I386_SUPPORTED=0
  ENV ARM_SUPPORTED=1
  ENV MACOSX_DEPLOYMENT_TARGET=$target_sdk_version
  RUN make wrapper -j

wrapper.clang:
  ARG --required sdk_version
  ARG --required kernel_version
  ARG --required target_sdk_version
  FROM +wrapper --sdk_version=$sdk_version --kernel_version=$kernel_version --target_sdk_version=$target_sdk_version
  ARG compilers=clang clang++
  FOR compiler IN $compilers
    ENV TARGETCOMPILER=$compiler
    ENV MACOSX_DEPLOYMENT_TARGET=$target_sdk_version
    RUN ./build_wrapper.sh
  END
  SAVE ARTIFACT /workspace/target/*

wrapper.gcc:
  ARG --required sdk_version
  ARG --required kernel_version
  ARG --required target_sdk_version
  FROM +wrapper --sdk_version=$sdk_version --kernel_version=$kernel_version --target_sdk_version=$target_sdk_version
  ARG compilers=gcc g++ gfortran
  FOR compiler IN $compilers
    ENV TARGETCOMPILER=$compiler
    ENV MACOSX_DEPLOYMENT_TARGET=$target_sdk_version
    RUN ./build_wrapper.sh
  END
  SAVE ARTIFACT /workspace/target/*

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
  ARG --required target_sdk_version
  ARG triple=$architecture-apple-darwin$kernel_version
  FROM +deps
  RUN apt install -y gcc g++ zlib1g-dev libmpc-dev libmpfr-dev libgmp-dev flex file

  # TODO: this shouldn't be needed
  RUN apt-get install -y --force-yes llvm-dev libxml2-dev uuid-dev libssl-dev bash patch make tar xz-utils bzip2 gzip sed cpio libbz2-dev zlib1g-dev

  IF [ $architecture = "aarch64" ]
    GIT CLONE --branch master-wip-apple-si https://github.com/iains/gcc-darwin-arm64 gcc
  ELSE IF [ $architecture = "x86_64" ]
    GIT CLONE --branch releases/gcc-12 https://github.com/gcc-mirror/gcc gcc
  ELSE
    RUN false
  END

  COPY (+wrapper.clang/ --kernel_version=$kernel_version --sdk_version=$sdk_version --target_sdk_version=$target_sdk_version) /osxcross
  ENV PATH=$PATH:/osxcross/bin
  COPY (+cctools/ --kernel_version=$kernel_version --target_sdk_version=$target_sdk_version) /cctools
  ENV PATH=$PATH:/cctools/bin

  COPY (+sdk/ --version=$sdk_version) /sdk
  RUN mkdir -p /osxcross/SDK
  RUN ln -s /sdk /osxcross/SDK/MacOSX$sdk_version.sdk

  # TODO: I think we can remove these
  COPY (+xar/ --target_sdk_version=$target_sdk_version) /sdk/usr
  COPY (+libtapi/ --target_sdk_version=$target_sdk_version) /sdk/usr

  COPY (+xar/lib --target_sdk_version=$target_sdk_version) /usr/local/lib
  COPY (+libtapi/lib --target_sdk_version=$target_sdk_version) /usr/local/lib
  RUN ldconfig

  # GCC requires that you build in a directory that is not a subdirectory of the source code
  # https://gcc.gnu.org/install/configure.html
  WORKDIR build

  # this being set is very important! we'll be building iphone binaries otherwise.
  ENV MACOSX_DEPLOYMENT_TARGET=$target_sdk_version
  RUN ../gcc/configure \
    --target=$triple \
    --with-sysroot=/sdk \
    --disable-nls \
    --enable-languages=c,c++,fortran,objc,obj-c++ \
    --without-headers \
    --enable-lto \
    --enable-checking=release \
    --disable-libstdcxx-pch \ # TODO: maybe enable this
    --prefix=/gcc \
    --with-system-zlib \
    --disable-multilib \
    --with-ld=/cctools/bin/$triple-ld \
    --with-as=/cctools/bin/$triple-as
  RUN make -j
  RUN make install
  SAVE ARTIFACT /gcc/*

image:
  ARG architectures=aarch64 x86_64
  ARG sdk_version=13.0
  ARG kernel_version=22
  ARG target_sdk_version=11
  FROM ubuntu:jammy
  COPY (+sdk/ --version=$sdk_version) /osxcross/SDK/MacOSX$sdk_version.sdk/
  RUN apt update
  # this is the clang we'll actually be using to compile stuff with!
  RUN apt install -y clang
  # for gcc
  RUN apt install -y libmpc-dev libmpfr-dev
  FOR architecture IN $architectures
    ENV triple=$architecture-apple-darwin$kernel_version
    COPY (+cctools/ --architecture=$architecture --sdk_version=$sdk_version --kernel_version=$kernel_version --target_sdk_version=$target_sdk_version) /cctools
    COPY (+gcc/ --architecture=$architecture --sdk_version=$sdk_version --kernel_version=$kernel_version --target_sdk_version=$target_sdk_version) /gcc
    COPY (+wrapper.clang/ --kernel_version=$kernel_version --sdk_version=$sdk_version --target_sdk_version=$target_sdk_version) /osxcross
    COPY (+wrapper.gcc/ --kernel_version=$kernel_version --sdk_version=$sdk_version --target_sdk_version=$target_sdk_version) /osxcross
    COPY (+gcc/lib --architecture=$architecture --sdk_version=$sdk_version --kernel_version=$kernel_version --target_sdk_version=$target_sdk_version) /usr/local/lib
    COPY (+gcc/include --architecture=$architecture --sdk_version=$sdk_version --kernel_version=$kernel_version --target_sdk_version=$target_sdk_version) /usr/local/include
    COPY (+gcc/$triple/lib --architecture=$architecture --sdk_version=$sdk_version --kernel_version=$kernel_version --target_sdk_version=$target_sdk_version) /usr/local/lib
    COPY (+gcc/$triple/include --architecture=$architecture --sdk_version=$sdk_version --kernel_version=$kernel_version --target_sdk_version=$target_sdk_version) /usr/local/include
  END

  COPY (+xar/lib --target_sdk_version=$target_sdk_version) /usr/local/lib
  COPY (+libtapi/lib --target_sdk_version=$target_sdk_version) /usr/local/lib
  RUN ldconfig

  ENV PATH=$PATH:/gcc/bin
  ENV PATH=$PATH:/cctools/bin
  ENV PATH=$PATH:/osxcross/bin
  WORKDIR /workspace
  SAVE IMAGE macos-cross-compiler

test:
  ARG architectures=aarch64 x86_64
  ARG sdk_version=13.0
  ARG kernel_version=22
  ARG target_sdk_version=11
  FROM +image
  COPY samples samples
  ENV MACOSX_DEPLOYMENT_TARGET=$target_sdk_version
  FOR architecture IN $architectures
    ENV triple=$architecture-apple-darwin$kernel_version
    RUN $triple-clang --target=$triple samples/hello.c -o hello-clang
    RUN $triple-clang++ --target=$triple samples/hello.cpp -o hello-clang++
    RUN $triple-gcc -L/osxcross/SDK/MacOSX$sdk_version.sdk/usr/lib/ \
      -I/osxcross/SDK/MacOSX$sdk_version.sdk/usr/include/ \
      samples/hello.c -o hello-gcc
    RUN $triple-g++ -L/osxcross/SDK/MacOSX$sdk_version.sdk/usr/lib/ \
      -I/osxcross/SDK/MacOSX$sdk_version.sdk/usr/include/ \
      samples/hello.cpp -o hello-g++
    # RUN $triple-gfortran samples/hello.f90 -o hello-gfortran
    # RUN $triple-rustc samples/hello.rs -o hello-rustc
  END
