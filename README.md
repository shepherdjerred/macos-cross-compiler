# macos-cross-compiler

## Requirements

* Docker
* [Earthly](https://earthly.dev/get-earthly)
* A copy of the macOS SDK in `/sdks`. See [the osxcross documentation about this](https://github.com/tpoechtrager/osxcross#packaging-the-sdk).

* <https://github.com/tpoechtrager/osxcross>
* <https://github.com/tpoechtrager/apple-libtapi>
* <https://github.com/tpoechtrager/cctools-port>
* <https://github.com/tpoechtrager/xar>
* <https://github.com/iains/gcc-darwin-arm64/>

## Usage

```bash
earthly +clang
earthly +gcc
```
