# Changelog

## 2024 12 07

- Re-add support for targeting x86_64

## 2024 12 04

- Update image from `jammy` -> `noble`
- Update `apple-libtapi` from `1100.0.11` -> `1300.6.5`
- Update `cctools` from `973.0.1` -> `986`
- Update `linker` from `609` -> `711`
- Update to latest version of `osxcross` (`29fe6dd35522073c9df5800f8cd1feb4b9a993a8`)
- Update from gcc `13` -> `14.2`
- Update macOS SDK from `13.0` -> `15.0`
- Update darwin kernel version from `22` -> `24`
- Remove support for targeting macOS x86_64
  - The above updates were failing to build on x86_64
  - With Apple deprecating x86_64 I felt it wasn't worth the hassle of updating
- Remove support for `zig-c++`
  - Seemed to have an issue with header search paths
  - I doubt anyone uses this; clang + gcc are enough for C++ and zig-cc is enough for Rust
