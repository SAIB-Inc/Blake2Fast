# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Blake2Fast is a high-performance implementation of BLAKE2 and BLAKE3 cryptographic hash functions for .NET. It provides:

- RFC 7693-compliant BLAKE2 implementations (BLAKE2b and BLAKE2s)
- BLAKE3 implementation 
- SIMD-accelerated implementations (SSE2 through AVX-512)
- Memory-efficient API using `Span<byte>` throughout

The library focuses on high speed and low memory usage while supporting multiple .NET frameworks from .NET 4.6 through .NET 9.0.

## Build Commands

```bash
# Standard build
dotnet build src/Blake2Fast -c Release

# Distribution build with versioning
dotnet build src/Blake2Fast -c Dist --version-suffix [version]

# Create NuGet package
dotnet build src/Blake2Fast -c Dist
# Package will be output to /out/nuget/
```

## Test Commands

```bash
# Run all tests
dotnet test tests/Blake.Test

# Run tests with coverage
dotnet test tests/Blake.Test -c Coverage

# Run tests with specific SIMD restrictions
dotnet test tests/Blake.Test -f net6.0 -e DOTNET_EnableAVX=0      # No AVX
dotnet test tests/Blake.Test -f net6.0 -e DOTNET_EnableHWIntrinsic=0  # No SIMD
```

## Benchmark Commands

```bash
# Run performance benchmarks
dotnet run -p tests/Blake.Bench
```

## Code Architecture

### Core Components

1. **Algorithm Variants**:
   - `Blake2b` (64-bit operations, 1-64 byte digests)
   - `Blake2s` (32-bit operations, 1-32 byte digests)
   - `Blake3` (newer algorithm with streaming support)

2. **API Structure**:
   - Static facade classes (`Blake2b`, `Blake2s`, `Blake3`) provide entry points
   - `IBlakeIncremental` interface for incremental hashing operations
   - Hash state structs handle the internal state of hash operations

3. **Implementation Structure**:
   - Scalar implementations (for all platforms)
   - SIMD-accelerated implementations (SSE4/SSSE3, AVX2, AVX-512)
   - Runtime detection and selection of optimal implementation

### Code Generation

The project uses T4 templates extensively:
- `.tt` files are templates
- `.g.cs` files are generated from templates
- This enables generating optimized variants while maintaining DRY principles
- Templates in `_Blake*.ttinclude` files define common patterns

### Cross-Platform Support

- Multiple target frameworks via conditional compilation
- `HWINTRINSICS` defined for platforms with hardware intrinsics support
- `BUILTIN_SPAN` for platforms with built-in Span support
- Fallback implementations for platforms without specific capabilities

## Development Guidelines

1. When adding features:
   - Maintain the existing API patterns across algorithms
   - Implement for all three algorithm variants when applicable
   - Consider both scalar and SIMD implementations

2. When modifying template-generated code:
   - Edit the `.tt` template files, not the generated `.g.cs` files
   - Ensure template changes generate proper code for all variants

3. For performance testing:
   - Use the benchmark project to validate performance impacts
   - Test with different SIMD capabilities to ensure proper fallbacks