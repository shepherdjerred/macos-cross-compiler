# macos-cross-compiler

This project allows you to compile C/C++ code on Linux that will be executed on macOS.

## Quick Start

### Requirements

* Docker
* [Earthly](https://earthly.dev/get-earthly)
* A copy of the macOS SDK in `/sdks`. See [the osxcross documentation about this](https://github.com/tpoechtrager/osxcross#packaging-the-sdk).

### Usage

```bash
earthly +image
```

## Compatability

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

It supports up to macOS 13 Ventura.

**Note:** This project is tested on modern verisons of macOS. It has not been tested with versions of macOS earlier than macOS 11 Big Sur.

## Technical Details

This repository is essentially a nice wrapper around the following projects:

* <https://github.com/tpoechtrager/apple-libtapi>
* <https://github.com/tpoechtrager/cctools-port>
* <https://github.com/tpoechtrager/xar>
* <https://github.com/iains/gcc-darwin-arm64/>

## Inspiration

This project would not have been possible without the [osxcross](https://github.com/tpoechtrager/osxcross) project.
