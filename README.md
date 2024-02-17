# macOS Cross Compiler

![A picture of Tux holding a red apple](./assets/256.png)

This project allows you to cross-compile code on Linux that will be executed on macOS.

It supports:

* ✅ C
* ✅ C++
* ✅ Fortran
* ✅ Rust

> [!NOTE]
> This project is focused on supporting newer versions of macOS and C, C++, Fortran, and Rust. Versions older than macOS 13 (Ventura) are not well tested, though they _should_ work fine.

## Quick Start

Install the requirements below, then follow the instructions in the usage section.

### Host Requirements

* Docker

### Usage

> [!IMPORTANT]
> The Docker image is quite large. It includes several compilers and the macOS SDK.

```bash
# Start a Docker container using the Docker image.
# Replace `$PWD/samples` with the path to the source you want to compile.
docker run \
  -v $PWD/samples:/workspace \
  --rm \
  -it \
  ghcr.io/shepherdjerred/macos-cross-compiler \
  /bin/bash

# Now that you're inside of the Docker container, you can run the compilers.

# Compile using gcc
## targeting darwin arm64
aarch64-apple-darwin22-gcc hello.c -o hello
aarch64-apple-darwin22-g++ hello.cpp -o hello
## targeting darwin x86_64
x86_64-apple-darwin22-gcc hello.c -o hello
x86_64-apple-darwin22-g++ hello.cpp -o hello

# Compile using clang
## for darwin arm64
aarch64-apple-darwin22-clang --target=aarch64-apple-darwin22 hello.c -o hello
aarch64-apple-darwin22-clang --target=aarch64-apple-darwin22 hello.cpp -o hello
## for darwin x86_64
x86_64-apple-darwin22-clang --target==x86_64-apple-darwin22 hello.c -o hello
x86_64-apple-darwin22-clang --target==x86_64-apple-darwin22 hello.cpp -o hello

# Compile using gfortran
## for darwin arm64
aarch64-apple-darwin22-gfortran hello.f90 -o hello
## for darwin x86_64
x86_64-apple-darwin22-gfortran hello.f90 -o hello

# Compile using Zig
## C targeting darwin arm64 (change aarch64 -> x86_64 to target amd64)
zig cc \
    -target aarch64-macos \
    --sysroot=/sdk \
    -I/sdk/usr/include \
    -L/sdk/usr/lib \
    -F/sdk/System/Library/Frameworks \
    -framework CoreFoundation \
    -o hello hello.c

## C++ targeting darwin arm64(change aarch64 -> x86_64 to target amd64)
zig c++ \
    -target aarch64-macos \
    --sysroot=/sdk -I/sdk/usr/include \
    -I/sdk/usr/include/c++/v1/ \
    -L/sdk/usr/lib \
    -lc++ \
    -F/sdk/System/Library/Frameworks \
    -framework CoreFoundation \
    -o hello hello.cpp

## Rust targeting darwin arm64 (change aarch64 -> x86_64 to target amd64)
export CC=zig-cc-aarch64-macos
cd rust && cargo build --target aarch64-apple-darwin
```

### Rust

Support for Rust requires a bit of project configuration.

```toml
# .cargo/config.toml
[build]
[target.aarch64-apple-darwin]
linker = "zig-cc-aarch64-macos"

[target.x86_64-apple-darwin]
linker = "zig-cc-x86_64-macos"
```

Once configured, you can run `cargo` after setting the `CC` variable:

```bash
export CC="zig-cc-x86_64-macos"
cargo build --target x86_64-apple-darwin

export CC="zig-cc-aarch64-macos"
cargo build --target aarch64-apple-darwin
```

### C, C++ and Fortran Compiler Executables

The table below shows the name of the executable for each architecture/compiler pair.

> [!NOTE]
> By default the target kernel version is `darwin22`. You'll need to change `darwin22` if you choose to compile for another kernel version.

|          | x86_64                         | aarch64                         |
|----------|--------------------------------|---------------------------------|
| **clang**    | x86_64-apple-darwin22-clang    | aarch64-apple-darwin22-clang    |
| **clang++**  | x86_64-apple-darwin22-clang++  | aarch64-apple-darwin22-clang++  |
| **gcc**      | x86_64-apple-darwin22-gcc      | aarch64-apple-darwin22-gcc      |
| **g++**      | x86_64-apple-darwin22-g++      | aarch64-apple-darwin22-g++      |
| **gfortran** | x86_64-apple-darwin22-gfortran | aarch64-apple-darwin22-gfortran |

The relevant compilers are located at `/osxcross/bin` and `/gcc/bin`. Both these directories are already on the `PATH` in the Docker container.

### cctools

This project compiles [cctools](https://github.com/tpoechtrager/cctools-port), which is Apple's version of [binutils](https://www.gnu.org/software/binutils/). These programs are low-level utilities that are used by compilers, such as the archiver `ar`, the loader `ld`, and the assembler `as`.

You probably don't need to run these programs directly, but if you do they are located at `/cctools/bin`, and they are also on the `PATH`.

Complete tool list:

* ObjectDump
* ar
* as
* bitcode_strip
* check_dylib
* checksyms
* cmpdylib
* codesign_allocate
* ctf_insert
* dyldinfo
* install_name_tool
* ld
* libtool
* lipo
* machocheck
* makerelocs
* mtoc
* mtor
* nm
* nmedit
* otool
* pagestuff
* ranlib
* redo_prebinding
* seg_addr_table
* seg_hack
* segedit
* size
* strings
* strip
* unwinddump
* vtool

## Code Signing, Notarizing, and Universal Binaries

[Code signing](https://developer.apple.com/documentation/security/code_signing_services) (but _not_ [notarizing](https://developer.apple.com/documentation/security/notarizing_macos_software_before_distribution/)) should be possible with this project, but it is untested. Building [universal binaries](https://developer.apple.com/documentation/apple-silicon/building-a-universal-macos-binary) should also be possible, but again, this is not tested.

## Target Compatibility

This project can build for macOS on both x86_64 and aarch64 archtictures, regardless of the host architecture.

|              | Linux x86_64 | Linux arm64 |
|--------------|--------------|-------------|
| **macOS x86_64** | ✅            | ✅           |
| **macOS aarch64**  | ✅            | ✅           |

> [!NOTE]
> aarch64 is Apple's internal name for arm64. They're used interchangably, but aarch64 is more correct when referring to macOS on arm64.

This project supports the following languages:

* C (up to C 17)
* C++ (up to C++ 20)
* Fortran (up to Fortran 2018)
* Rust (any version)

This project supports the following versions of macOS:

* ✅ macOS 11 Big Sur
* ✅ macOS 12 Monterey
* ✅ macOS 13 Ventura

Support for macOS 14 Sonoma has not been extensively tested. macOS 14-specific features can be added by updating the SDK version. The Docker image uses the 13.0 SDK by default.

> [!IMPORTANT]
> This project is tested on modern verisons of macOS, Clang, and GCC. It has not been tested with older versions of these softwares. If you need compatabiltiy with older versions, check out the [osxcross project](https://github.com/tpoechtrager/osxcross).

## Technical Details

This repository is essentially a wrapper around the following projects:

* <https://github.com/tpoechtrager/apple-libtapi>
* <https://github.com/tpoechtrager/cctools-port>
* <https://github.com/tpoechtrager/xar>
* <https://github.com/iains/gcc-darwin-arm64>

These resources were helpful when working on this project:

* <https://www.lurklurk.org/linkers/linkers.html>
* <http://www.yolinux.com/TUTORIALS/LibraryArchives-StaticAndDynamic.html>
* <https://gist.github.com/loderunner/b6846dd82967ac048439>
* <http://clarkkromenaker.com/post/library-dynamic-loading-mac/>
* <https://github.com/qyang-nj/llios>

The Zig and Rust portion were informed by these resources:

* <https://andrewkelley.me/post/zig-cc-powerful-drop-in-replacement-gcc-clang.html>
* <https://actually.fyi/posts/zig-makes-rust-cross-compilation-just-work/>

## Development

The Docker images for this repository are built with [Earthly](https://earthly.dev).

```bash
# Create a Docker image tagged as `shepherdjerred/macos-cross-compiler`
# The first run will take ~20 minutes on an M1 MacBook.
# Subsequent runs are faster.
earthly +image

# Verify that the compilers work correctly
earthly +test

# If you're on macOS, try actually running the binaries
earthly +validate
```

## Inspiration

This project would not have been possible without the [osxcross](https://github.com/tpoechtrager/osxcross) project.
