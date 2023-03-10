# macOS Cross Compiler

This project allows you to compile C/C++ code on Linux that will be executed on macOS. This project is focused on supporting newer versions of macOS and C/C++. Older versions are not well tested.

## Quick Start

Install the requirements below, then follow the instructions in the usage section.

### Requirements

* [Docker](https://docs.docker.com/engine/install/)
* [Earthly](https://earthly.dev/get-earthly)
  * Earthly is a combination of Docker and [GNU Make](https://www.gnu.org/software/make/). It builds everything in Docker containers, which makes it easy to automatically cache and parallelize builds.
* A copy of the macOS SDK in `/sdks`. See [the osxcross documentation about this](https://github.com/tpoechtrager/osxcross#packaging-the-sdk).

### Usage

```bash
# Create a Docker image tagged as `macos-cross-compiler`
# The first run will take ~20 minutes on an M1 MacBook.
# Subsequent runs are faster.
earthly +image

# Verify that the compilers work correctly
earthly +test

# Start a Docker container using the image we built earlier
# Replace this with the path to the source you want to compile
SOURCE_CODE=$PWD/samples
docker run -v $SOURCE_CODE:/workspace/code macos-cross-compiler

# Inside of the Docker container

# Set the minimum version of macOS you want to target
export MACOSX_DEPLOYMENT_TARGET=13

# Compile something using gcc
## for arm64
aarch64-apple-darwin22-gcc samples/hello.c -o hello
## for x86_64
x86_64-apple-darwin-gcc samples/hello.c -o hello

# Compile using clang
## for arm64
aarch64-apple-darwin22-clang samples/hello.c -o hello
## for x86_64
x86_64-apple-darwin22-clang samples/hello.c -o hello

# Compile using gfortran
## for arm64
aarch64-apple-darwin22-gfortran samples/hello.c -o hello
## for x86_64
x86_64-apple-darwin22-gfortran samples/hello.c -o hello
```

### Compiler Executables

The table below shows the name of the executable for each architecture/compiler pair.

**Note:** By default the target kernel version is `darwin22`. You'll need to change `darwin22` if you choose to compile for another kernel version.

|          | x86_64                         | aarch64                         |
|----------|--------------------------------|---------------------------------|
| **clang**    | x86_64-apple-darwin22-clang    | aarch64-apple-darwin22-clang    |
| **gcc**      | x86_64-apple-darwin22-gcc      | aarch64-apple-darwin22-gcc      |
| **clang++**  | x86_64-apple-darwin22-clang++  | aarch64-apple-darwin22-clang++  |
| **g++**      | x86_64-apple-darwin22-g++      | aarch64-apple-darwin22-g++      |
| **gfortran** | x86_64-apple-darwin22-gfortran | aarch64-apple-darwin22-gfortran |

The relevant compilers are located at `/osxcross/bin` and `/gcc/bin`. Both these directories are already on the `PATH` in the Docker container.

### cctools

This project compiles [cctools](https://github.com/tpoechtrager/cctools-port), which is Apple's version of [binutils](https://www.gnu.org/software/binutils/). These programs are low-level utilities that are used by compilers, such as the archiver `ar`, the loader `ld`, and the assembler `as`.

You probably don't need to run these programs directly, but if you do they are located at `/cctools/bin`, and they are also on the `PATH`.

Tool list:

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

## Compatibility

This project can build for macOS on both x86_64 and aarch64 archtictures, regardless of the host architecture.

|              | Linux x86_64 | Linux arm64 |
|--------------|--------------|-------------|
| **macOS x86_64** | ???            | ???           |
| **macOS aarch64**  | ???            | ???           |

**Note:** aarch64 is Apple's internal name for arm64. They're used interchangably, but aarch64 is more correct when referring to macOS on arm64.

This project supports the following languages:

* C (up to C 17)
* C++ (up to C++ 20)
* Fortran (up to Fortran 2018) (needs confirmation)

This project supports the following versions of macOS:

* ??? macOS 11 Big Sur
* ??? macOS 12 Monterey
* ??? macOS 13 Ventura

**Note:** This project is tested on modern verisons of macOS, Clang, and GCC. It has not been tested with older versions of these softwares. If you need compatabiltiy with older versions, check out the [osxcross project](https://github.com/tpoechtrager/osxcross).

## Technical Details

This repository is essentially a wrapper around the following projects:

* <https://github.com/tpoechtrager/apple-libtapi>
* <https://github.com/tpoechtrager/cctools-port>
* <https://github.com/tpoechtrager/xar>
* <https://github.com/iains/gcc-darwin-arm64>

These resources were helpful when working on this project:

* https://www.lurklurk.org/linkers/linkers.html
* http://www.yolinux.com/TUTORIALS/LibraryArchives-StaticAndDynamic.html
* https://gist.github.com/loderunner/b6846dd82967ac048439
* http://clarkkromenaker.com/post/library-dynamic-loading-mac/
* https://github.com/qyang-nj/llios

## Inspiration

This project would not have been possible without the [osxcross](https://github.com/tpoechtrager/osxcross) project.
