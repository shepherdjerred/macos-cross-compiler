VERSION 0.8
FROM ubuntu:noble
WORKDIR /workspace
ARG --global cores=16

ci:
  # TODO: build for arm64 too
  BUILD --platform=linux/amd64 +image
  BUILD +test

deps:
  RUN apt update -y
  RUN apt install -y build-essential cmake clang git

xar:
  FROM +deps
  ARG --required target_sdk_version
  RUN apt install -y libxml2-dev libssl-dev zlib1g-dev
  GIT CLONE --branch 5fa4675419cfec60ac19a9c7f7c2d0e7c831a497 https://github.com/tpoechtrager/xar .
  WORKDIR xar
  ENV MACOSX_DEPLOYMENT_TARGET=$target_sdk_version
  RUN ./configure --prefix=/xar
  RUN make -j$cores
  RUN make install
  SAVE ARTIFACT /xar/*

libdispatch:
  FROM +deps
  ARG --required target_sdk_version
  ARG version=fdf3fc85a9557635668c78801d79f10161d83f12
  GIT CLONE --branch $version https://github.com/tpoechtrager/apple-libdispatch .
  ENV MACOSX_DEPLOYMENT_TARGET=$target_sdk_version
  ENV TARGETDIR=/libdispatch
  RUN mkdir -p build
  WORKDIR build
  RUN CC=clang CXX=clang++ cmake .. -DCMAKE_BUILD_TYPE=RELEASE -DCMAKE_INSTALL_PREFIX=$TARGETDIR
  RUN make install -j$cores
  SAVE ARTIFACT /libdispatch/*

libtapi:
  FROM +deps
  RUN apt install -y python3
  ARG --required target_sdk_version
  ARG version=1300.6.5
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
  ARG cctools_version=1010.6
  ARG linker_verison=951.9
  GIT CLONE --branch $cctools_version-ld64-$linker_verison https://github.com/tpoechtrager/cctools-port .
  COPY (+xar/ --target_sdk_version=$target_sdk_version) /xar
  COPY (+libtapi/ --target_sdk_version=$target_sdk_version) /libtapi
  COPY (+libdispatch/ --target_sdk_version=$target_sdk_version) /libdispatch
  WORKDIR cctools
  ENV MACOSX_DEPLOYMENT_TARGET=$target_sdk_version
  RUN ./configure \
    --prefix=/cctools \
    --with-libtapi=/libtapi \
    --with-libxar=/libxar \
    --with-libdispatch=/libdispatch \
    --with-libblocksruntime=/libdispatch \
    --target=$triple
  # now that we've tricked autoconf by pretending to build for arm, let's _actually_ build for arm64
  # https://github.com/tpoechtrager/cctools-port/issues/6
  RUN find . -name Makefile -print0 | xargs -0 sed -i "s/arm-apple-darwin$kernel_version/arm64-apple-darwin$kernel_version/g"
  RUN make -j$cores
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
  GIT CLONE --branch=29fe6dd35522073c9df5800f8cd1feb4b9a993a8 https://github.com/tpoechtrager/osxcross .
  WORKDIR wrapper
  # this is in build.sh in osxcross
  ENV VERSION=1.5
  ENV SDK_VERSION=$sdk_version
  ENV TARGET=darwin$kernel_version
  # this needs to match the version of the linker in cctools
  ENV LINKER_VERSION=951.9
  ENV X86_64H_SUPPORTED=0
  ENV I386_SUPPORTED=0
  ENV ARM_SUPPORTED=1
  ENV MACOSX_DEPLOYMENT_TARGET=$target_sdk_version
  RUN make wrapper -j$cores

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

sdk.download:
  RUN apt update
  RUN apt install -y wget
  ARG --required version
  RUN wget https://github.com/joseluisq/macosx-sdks/releases/download/$version/MacOSX$version.sdk.tar.xz
  SAVE ARTIFACT MacOSX$version.sdk.tar.xz

sdk:
  ARG --required version
  ARG --required download_sdk
  FROM +deps
  IF [ $download_sdk = "true" ]
    COPY (+sdk.download/MacOSX$version.sdk.tar.xz --version=$version) .
    RUN tar -xf MacOSX$version.sdk.tar.xz
    RUN mv MacOSX*.sdk MacOSX$version.sdk || true # newer versions of the SDK don't need to be moved
  ELSE
    COPY sdks/MacOSX$version.sdk.tar.xz .
    RUN tar -xf MacOSX$version.sdk.tar.xz
  END
  SAVE ARTIFACT MacOSX$version.sdk/*

gcc:
  ARG --required download_sdk
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
    GIT CLONE --branch=gcc-14-2-darwin https://github.com/iains/gcc-14-branch gcc
  ELSE IF [ $architecture = "x86_64" ]
    GIT CLONE --branch=gcc-14-2-darwin https://github.com/iains/gcc-14-branch gcc
  ELSE
    RUN false
  END

  COPY (+wrapper.clang/ --kernel_version=$kernel_version --sdk_version=$sdk_version --target_sdk_version=$target_sdk_version) /osxcross
  ENV PATH=$PATH:/osxcross/bin
  COPY (+cctools/ --kernel_version=$kernel_version --target_sdk_version=$target_sdk_version) /cctools
  ENV PATH=$PATH:/cctools/bin

  COPY (+sdk/ --version=$sdk_version --download_sdk=$download_sdk) /sdk
  RUN mkdir -p /osxcross/SDK
  RUN ln -s /sdk /osxcross/SDK/MacOSX$sdk_version.sdk

  # TODO: I think we can remove these
  COPY (+xar/ --target_sdk_version=$target_sdk_version) /sdk/usr
  COPY (+libtapi/ --target_sdk_version=$target_sdk_version) /sdk/usr
  COPY (+libdispatch/ --target_sdk_version=$target_sdk_version) /sdk/usr

  COPY (+xar/lib --target_sdk_version=$target_sdk_version) /usr/local/lib
  COPY (+libtapi/lib --target_sdk_version=$target_sdk_version) /usr/local/lib
  COPY (+libdispatch/lib --target_sdk_version=$target_sdk_version) /usr/local/lib
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
  RUN make -j$cores
  RUN make install
  SAVE ARTIFACT /gcc/*

image:
  ARG architectures=aarch64 x86_64
  ARG sdk_version=15.0
  ARG kernel_version=24
  ARG target_sdk_version=11.0.0
  ARG download_sdk=true
  COPY (+sdk/ --version=$sdk_version --download_sdk=$download_sdk) /osxcross/SDK/MacOSX$sdk_version.sdk/
  RUN ln -s /osxcross/SDK/MacOSX$sdk_version.sdk/ /sdk
  RUN apt update
  # this is the clang we'll actually be using to compile stuff with!
  RUN apt install -y clang
  # for inspecting the binaries
  RUN apt install -y file
  # for gcc
  RUN apt install -y libmpc-dev libmpfr-dev

  # for rust
  COPY ./zig+zig/zig /usr/local/bin
  RUN apt install -y curl
  RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
  ENV PATH=$PATH:/root/.cargo/bin

  FOR architecture IN $architectures
    ENV triple=$architecture-apple-darwin$kernel_version
    COPY (+cctools/ --architecture=$architecture --sdk_version=$sdk_version --kernel_version=$kernel_version --target_sdk_version=$target_sdk_version) /cctools
    COPY (+gcc/ --architecture=$architecture --sdk_version=$sdk_version --kernel_version=$kernel_version --target_sdk_version=$target_sdk_version --download_sdk=$download_sdk) /gcc
    COPY (+wrapper.clang/ --kernel_version=$kernel_version --sdk_version=$sdk_version --target_sdk_version=$target_sdk_version) /osxcross
    COPY (+wrapper.gcc/ --kernel_version=$kernel_version --sdk_version=$sdk_version --target_sdk_version=$target_sdk_version) /osxcross
    COPY (+gcc/lib --architecture=$architecture --sdk_version=$sdk_version --kernel_version=$kernel_version --target_sdk_version=$target_sdk_version --download_sdk=$download_sdk) /usr/local/lib
    COPY (+gcc/include --architecture=$architecture --sdk_version=$sdk_version --kernel_version=$kernel_version --target_sdk_version=$target_sdk_version --download_sdk=$download_sdk) /usr/local/include
    COPY (+gcc/$triple/lib --architecture=$architecture --sdk_version=$sdk_version --kernel_version=$kernel_version --target_sdk_version=$target_sdk_version --download_sdk=$download_sdk) /usr/local/lib
    COPY (+gcc/$triple/include --architecture=$architecture --sdk_version=$sdk_version --kernel_version=$kernel_version --target_sdk_version=$target_sdk_version --download_sdk=$download_sdk) /usr/local/include
    COPY ./zig/zig-cc-$architecture-macos /usr/local/bin/
    RUN rustup target add $architecture-apple-darwin
  END

  COPY (+xar/lib --target_sdk_version=$target_sdk_version) /usr/local/lib
  COPY (+libtapi/lib --target_sdk_version=$target_sdk_version) /usr/local/lib
  COPY (+libdispatch/lib --target_sdk_version=$target_sdk_version) /usr/local/lib
  RUN ldconfig

  ENV PATH=$PATH:/usr/local/bin
  ENV PATH=$PATH:/gcc/bin
  ENV PATH=$PATH:/cctools/bin
  ENV PATH=$PATH:/osxcross/bin
  ENV MACOSX_DEPLOYMENT_TARGET=$target_sdk_version
  WORKDIR /workspace
  SAVE IMAGE --push ghcr.io/shepherdjerred/macos-cross-compiler:latest
  SAVE IMAGE --push ghcr.io/shepherdjerred/macos-cross-compiler:$sdk_version

test:
  ARG architectures=aarch64 x86_64
  ARG sdk_version=15.0
  ARG kernel_version=24
  ARG target_sdk_version=11.0.0
  ARG download_sdk=true
  FROM +image --architectures=$architectures --sdk_version=$sdk_version --kernel_version=$kernel_version --target_sdk_version=$target_sdk_version --download_sdk=$download_sdk
  COPY ./samples/ samples/
  FOR architecture IN $architectures
    ENV triple=$architecture-apple-darwin$kernel_version

    RUN mkdir -p out/

    # compile the samples
    RUN $triple-clang --target=$triple samples/hello.c -o out/hello-clang
    RUN $triple-clang++ --target=$triple samples/hello.cpp -o out/hello-clang++
    RUN $triple-gcc samples/hello.c -o out/hello-gcc
    RUN $triple-g++ samples/hello.cpp -o out/hello-g++
    RUN $triple-gfortran samples/hello.f90 -o out/hello-gfortran
    RUN zig cc \
      -target $architecture-macos \
      --sysroot=/sdk \
      -I/sdk/usr/include \
      -L/sdk/usr/lib \
      -F/sdk/System/Library/Frameworks \
      -framework CoreFoundation \
      -o out/hello-zig-c samples/hello.c
    ENV CC="zig-cc-$architecture-macos"
    RUN cd samples/rust && cargo build --target $architecture-apple-darwin && mv target/$architecture-apple-darwin/debug/hello ../../out/hello-rust

    # verify that the cross-compiler targeted the correct architecture
    IF [ "$architecture" = "aarch64" ]
      RUN file out/hello-clang | grep -q "Mach-O 64-bit arm64 executable"
      RUN file out/hello-clang++ | grep -q "Mach-O 64-bit arm64 executable"
      RUN file out/hello-gcc | grep -q "Mach-O 64-bit arm64 executable"
      RUN file out/hello-g++ | grep -q "Mach-O 64-bit arm64 executable"
      RUN file out/hello-gfortran | grep -q "Mach-O 64-bit arm64 executable"
      RUN file out/hello-zig-c | grep -q "Mach-O 64-bit arm64 executable"
      RUN file out/hello-rust | grep -q "Mach-O 64-bit arm64 executable"
    ELSE
      RUN file out/hello-clang | grep -q "Mach-O 64-bit $architecture executable"
      RUN file out/hello-clang++ | grep -q "Mach-O 64-bit $architecture executable"
      RUN file out/hello-gcc | grep -q "Mach-O 64-bit $architecture executable"
      RUN file out/hello-g++ | grep -q "Mach-O 64-bit $architecture executable"
      RUN file out/hello-gfortran | grep -q "Mach-O 64-bit $architecture executable"
      RUN file out/hello-zig-c | grep -q "Mach-O 64-bit $architecture executable"
      RUN file out/hello-rust | grep -q "Mach-O 64-bit $architecture executable"
    END

    SAVE ARTIFACT out/* AS LOCAL out/$architecture/
  END

# Can only be run on macOS
validate:
  LOCALLY

  ARG USERARCH
  LET arch = $USERARCH
  # convert arm64 -> aarch64
  IF [ $arch = "arm64" ]
    SET arch=aarch64
  END
  # convert x86_64 -> amd64
  IF [ $arch = "x86_64" ]
    SET arch=amd64
  END

  WAIT
    BUILD +test
  END

  RUN ./out/$arch/hello-clang
  RUN ./out/$arch/hello-clang++
  RUN ./out/$arch/hello-g++
  RUN ./out/$arch/hello-gcc
  # note: required fortran to be installed on your macOS device
  RUN ./out/$arch/hello-gfortran
  RUN ./out/$arch/hello-zig-c
  RUN ./out/$arch/hello-rust
