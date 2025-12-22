# BLAS/LAPACK on Windows: Solutions

## Status: IMPLEMENTED

BLAS/LAPACK is now **opt-in** (removed from default features). Three backend options available.

## Root Cause (Historical)

When building with `--all-features` or when `blas-lapack` was enabled (previously in default features), the build failed on Windows because:

1. `runmat` crate has `default = ["gui", "blas-lapack", "wgpu"]`
2. `blas-lapack` enables `runmat-runtime/blas-lapack`
3. On non-macOS platforms, this pulls in `openblas-src` with `features = ["system", "cblas"]`
4. The `system` feature requires OpenBLAS installed via **vcpkg** (or system packages on Linux)

```
runmat (default features)
  └── blas-lapack
        └── runmat-runtime/blas-lapack
              └── openblas-src (system, cblas) ← Requires vcpkg on Windows!
```

**Key insight**: On Windows, `openblas-src` cannot build from source. It MUST use precompiled binaries via vcpkg.

---

## Solution Options

### Option 1: Default Build (No BLAS) ✅ RECOMMENDED

BLAS is now opt-in. Default build works out of the box:

```bash
cargo build                    # Just works!
cargo test -p runmat-turbine   # Just works!
cargo clippy -- -D warnings    # Just works!
```

### Option 2: Intel MKL (Auto-Download) ✅ EASY

Intel MKL auto-downloads precompiled binaries (~200MB). No manual setup needed:

```bash
cargo build --features blas-mkl
```

### Option 3: OpenBLAS via vcpkg (Manual Setup)

```powershell
# One-time setup
git clone https://github.com/Microsoft/vcpkg.git C:\vcpkg
cd C:\vcpkg
.\bootstrap-vcpkg.bat
.\vcpkg integrate install

# Install OpenBLAS (dynamic linking)
.\vcpkg install openblas:x64-windows

# Or for static linking
.\vcpkg install openblas:x64-windows-static-md
```

Then set environment variable:
```powershell
$env:VCPKG_ROOT = "C:\vcpkg"
```

---

## Summary of Changes

| File | Change |
|------|--------|
| `runmat/Cargo.toml` | Removed `blas-lapack` from defaults, added `blas-mkl` feature |
| `crates/runmat-runtime/Cargo.toml` | Added `intel-mkl-src` dependency and `blas-mkl` feature |

## Feature Comparison

| Feature | Backend | Windows Setup | Performance |
|---------|---------|---------------|-------------|
| (none) | No BLAS | None needed | Basic |
| `blas-mkl` | Intel MKL | Auto-download | Excellent |
| `blas-lapack` | OpenBLAS | vcpkg required | Good |

---

## References

- [openblas-src crate](https://crates.io/crates/openblas-src)
- [vcpkg OpenBLAS port](https://github.com/microsoft/vcpkg/tree/master/ports/openblas)
- [Intel MKL for Rust](https://crates.io/crates/intel-mkl-src)
