# macOS Cross-Compiler Dagger Module

This Dagger module builds a macOS cross-compiler supporting multiple architectures and compilers.

## Features

- **Multi-architecture support**: aarch64 (ARM64) and x86_64
- **Multiple compilers**: Clang, GCC, Zig
- **Language support**: C, C++, Objective-C, Objective-C++, Fortran, Rust
- **Automated testing**: Validates cross-compiled binaries
- **Container registry**: Automatic image publishing to GitHub Container Registry

## Usage

### Prerequisites

- [Dagger CLI](https://dagger.io/install)
- [Bun runtime](https://bun.sh/docs/installation)

### Install dependencies

```bash
bun install
```

### Build the cross-compiler

```bash
dagger call build-image --source=..
```

### Run tests

```bash
dagger call test --source=..
```

### Run full CI pipeline

```bash
dagger call ci --source=..
```

### Push to registry (requires credentials)

```bash
dagger call ci \
  --source=.. \
  --ghcr-username="your-username" \
  --ghcr-password="env://GHCR_PASSWORD"
```

### Validate executables (macOS only)

```bash
dagger call validate --source=..
```

## Configuration

The module supports the following parameters:

- `architectures`: Comma-separated list of target architectures (default: "aarch64,x86_64")
- `sdkVersion`: macOS SDK version (default: "15.0")
- `kernelVersion`: Darwin kernel version (default: "24")
- `targetSdkVersion`: Target SDK version for deployment (default: "11.0.0")
- `downloadSdk`: Whether to download SDK or use local copy (default: true)
- `cores`: Number of CPU cores for compilation (default: 16)

## Architecture

The build process follows this sequence:

1. **Base dependencies**: Ubuntu Noble with build tools
2. **Support libraries**: xar, libdispatch, libtapi
3. **SDK preparation**: Download or use local macOS SDK
4. **Cross-compilation tools**: cctools, linker, assembler
5. **Compiler wrappers**: osxcross wrappers for clang and GCC
6. **Zig compiler**: For additional Rust support
7. **GCC compilation**: Full GCC cross-compiler build
8. **Final image**: Combined image with all tools and libraries

## Output

The final container image includes:

- Cross-compilation toolchain for specified architectures
- Clang, GCC, and Zig compilers
- Rust support via zig-cc
- All necessary libraries and SDKs
- Test sample programs

## Testing

The module includes comprehensive tests that:

- Compile sample programs with all supported compilers
- Verify correct target architecture
- Export test artifacts for inspection
- Validate executables on macOS (when available)

## Integration

This module integrates with GitHub Actions for continuous integration and automatic image publishing to GitHub Container Registry.
