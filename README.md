# macOS Cross Compiler

This project allows you to compile C/C++ code on Linux that will be executed on macOS.

## Quick Start

Install the requirements, then follow the instructions in the Usage section.

### Requirements

* [Docker](https://docs.docker.com/engine/install/)
* [Earthly](https://earthly.dev/get-earthly)
  * Earthly is a combination of Docker and [GNU Make](https://www.gnu.org/software/make/). It builds everything in Docker containers, which makes it easy to automatically cache and parallelize builds.
* A copy of the macOS SDK in `/sdks`. See [the osxcross documentation about this](https://github.com/tpoechtrager/osxcross#packaging-the-sdk).

### Usage

```bash
# Create a Docker image tagged as `macos-cross-compiler`
earthly +image
```

## Compatibility

This project can build for macOS on both x86_64 and arm64 archtictures, regardless of the host architecture.

|              | Linux x86_64 | Linux arm64 |
|--------------|--------------|-------------|
| macOS x86_64 | ✅            | ✅           |
| macOS arm64  | ✅            | ✅           |

It supports the following languages:

* C (up to C17)
* C++ (up to C++ 20)
* Objective C
* Objective C++
* Fortran (up to Fortran 2018)

It supports the following versions of macOS:

* [x] macOS 11 Big Sur
* [x] macOS 12 Monterey
* [x] macOS 13 Ventura

**Note:** This project is tested on modern verisons of macOS, Clang, and GCC. It has not been tested with older versions of these softwares. If you need compatabiltiy with older versions, check out the [osxcross project](https://github.com/tpoechtrager/osxcross).

## Technical Details

This repository is essentially a nice wrapper around the following projects:

* <https://github.com/tpoechtrager/apple-libtapi>
* <https://github.com/tpoechtrager/cctools-port>
* <https://github.com/tpoechtrager/xar>
* <https://github.com/iains/gcc-darwin-arm64>

## Inspiration

This project would not have been possible without the [osxcross](https://github.com/tpoechtrager/osxcross) project.
