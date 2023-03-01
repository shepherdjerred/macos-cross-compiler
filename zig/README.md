# macOS Cross Compiler (via Zig)

## Quick Start

```bash
# C and C++
earthly +image --sdk_version=13.0 --architecture=aarch64

cd ..

docker run -it --rm \
  -v $(pwd)/samples:/samples \
  macos-cross-compiler:zig \
  /bin/bash

cd /samples

# C
zig cc \
    -target aarch64-macos \
    --sysroot=/sdk \
    -I/sdk/usr/include \
    -L/sdk/usr/lib \
    -F/sdk/System/Library/Frameworks \
    -framework CoreFoundation \
    -o hello hello.c

# C++
zig c++ \
    -target aarch64-macos \
    --sysroot=/sdk -I/sdk/usr/include \
    -I/sdk/usr/include/c++/v1/ \
    -L/sdk/usr/lib \
    -lc++ \
    -F/sdk/System/Library/Frameworks \
    -framework CoreFoundation \
    -o hello hello.cpp

# Rust
earthly +image.rust --sdk_version=13.0 --architecture=aarch64

cd ..

docker run -it --rm \
  -v $(pwd)/samples:/samples \
  macos-cross-compiler:zig-rust \
  /bin/bash

cd /samples/rust
rustup target add aarch64-apple-darwin
export CC=zig-cc-aarch64-macos
cargo build --target aarch64-apple-darwin
```

## Resources

<https://andrewkelley.me/post/zig-cc-powerful-drop-in-replacement-gcc-clang.html>
<https://actually.fyi/posts/zig-makes-rust-cross-compilation-just-work/>
