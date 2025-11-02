# Bazel Build System

This document describes how to build and use the Envoy Data Plane API with Bazel.

## Overview

The Envoy Data Plane API supports two build modes:
- **bzlmod** (recommended): Modern Bazel module system (Bazel 6.0+)
- **WORKSPACE**: Legacy build system (maintained for backwards compatibility)

## Building with Bzlmod (Recommended)

Bzlmod is the modern dependency management system for Bazel (Bazel 6.0+). It provides:
- Better dependency resolution
- Simplified dependency management  
- Native support in Bazel Central Registry (BCR)
- Improved hermiticity and reproducibility

### MODULE.bazel

The `MODULE.bazel` file defines all dependencies using modern bazel_dep declarations:

```starlark
module(
    name = "envoy_api",
    version = "0.0.0-dev",
)

# Dependencies from Bazel Central Registry
bazel_dep(name = "abseil-cpp", version = "20250127.1", repo_name = "com_google_absl")
bazel_dep(name = "protobuf", version = "29.3", repo_name = "com_google_protobuf")
bazel_dep(name = "grpc", version = "1.66.0.bcr.2", repo_name = "com_github_grpc_grpc")
# ... more dependencies
```

### Key Features

1. **Latest BCR Versions**: Uses the most recent compatible versions from Bazel Central Registry
   - `bazel_skylib`: 1.8.2
   - `protobuf`: 29.3  
   - `abseil-cpp`: 20250127.1
   - `grpc`: 1.66.0.bcr.2
   - `googleapis`: 0.0.0-20241220-5e258e33.bcr.1

2. **Module Extensions**: For dependencies not in BCR
   - `non_module_deps`: Handles custom dependencies (prometheus, zipkin, cel, jsonschema)
   - `switched_rules`: Configures googleapis language support
   - `go_deps`: Manages Go proto dependencies

3. **Bzlmod Compatibility**: The codebase includes compatibility fixes for bzlmod
   - `api_build_system.bzl`: Uses `repo_name` instead of `workspace_name`
   - Automatic detection of bzlmod mode via `_IS_BZLMOD`

### Building

To build targets using bzlmod:

```bash
# Build all protos
bazel build //:all_protos

# Build v3 protos
bazel build //:v3_protos

# Build a specific package
bazel build //envoy/config/core/v3:pkg

# Query targets
bazel query //...
```

## Building with WORKSPACE (Legacy)

For backwards compatibility, WORKSPACE-based builds are still supported. The WORKSPACE file can be created by calling `api_dependencies()` from `bazel/repositories.bzl`.

### Creating a WORKSPACE

```python
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

# Load Envoy API
http_archive(
    name = "envoy_api",
    # ...
)

load("@envoy_api//bazel:repositories.bzl", "api_dependencies")
api_dependencies()
```

## Dependencies

### Dependencies from Bazel Central Registry (BCR)

These dependencies are available in BCR and used directly via `bazel_dep`:

- **abseil-cpp**: C++ common libraries
- **bazel_skylib**: Common Bazel utilities
- **gazelle**: BUILD file generator for Go
- **googleapis**: Google API definitions
- **googletest**: C++ testing framework
- **grpc**: gRPC framework
- **opencensus-proto**: OpenCensus protocol definitions
- **opentelemetry-proto**: OpenTelemetry protocol definitions
- **protobuf**: Protocol Buffers
- **protoc-gen-validate**: Protocol buffer validation
- **re2**: Regular expression library
- **rules_go**: Bazel rules for Go
- **rules_jvm_external**: Java dependency management
- **rules_proto**: Bazel rules for Protocol Buffers
- **rules_python**: Bazel rules for Python
- **xds**: xDS API definitions

### Dependencies via Module Extension

These dependencies are not yet in BCR or require custom build files:

- **prometheus_metrics_model**: Prometheus client model
- **com_github_openzipkin_zipkinapi**: Zipkin API definitions  
- **dev_cel**: Common Expression Language
- **com_github_chrusty_protoc_gen_jsonschema**: Proto to JSON schema compiler

## Migration from WORKSPACE to MODULE.bazel

If you're consuming this repository and want to migrate to bzlmod:

1. Create a `MODULE.bazel` file in your repository
2. Add `bazel_dep(name = "envoy_api", version = "...")` 
3. Remove the WORKSPACE definitions for envoy_api
4. Update any `@external_repo` references if needed (most remain the same)

### Example Consumer MODULE.bazel

```starlark
module(
    name = "my_project",
    version = "1.0.0",
)

bazel_dep(name = "envoy_api", version = "0.0.0-20250128-4de3c74")
```

## Published Versions

The envoy_api module is published to the Bazel Central Registry (BCR):
- Repository: https://github.com/bazelbuild/bazel-central-registry/tree/main/modules/envoy_api
- Latest versions can be found in the BCR

## Compatibility

- **Minimum Bazel Version**: 6.0.0 (for bzlmod support)
- **Recommended Bazel Version**: 8.0.0+
- **Legacy WORKSPACE**: Continues to work with Bazel 5.0+

## Development

### Testing Changes

```bash
# Build and test
bazel build //...
bazel test //test/...

# Query dependencies
bazel mod graph
bazel mod show_repo <repo_name>

# Clean build
bazel clean --expunge
```

### Adding New Dependencies

1. For BCR dependencies: Add `bazel_dep()` to MODULE.bazel
2. For non-BCR dependencies: Add to `non_module_deps` extension in `bazel/repositories.bzl`
3. Update `bazel/repository_locations.bzl` with version and metadata

## References

- [Bazel bzlmod Documentation](https://bazel.build/external/overview#bzlmod)
- [Bazel Central Registry](https://registry.bazel.build/)
- [Migration Guide](https://bazel.build/external/migration)
- [Envoy Proxy Repository](https://github.com/envoyproxy/envoy)
