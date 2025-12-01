# Building libgpod on macOS (BakBeat Meta-Fork)

This document describes how to build the BakBeat-focused meta-fork of libgpod on macOS. This fork is used as a C library dependency from the BakBeat Swift/ObjC app for reading and writing classic iPod iTunesDB files.

The build process produces a `.dylib` and headers installed into a local `_install/` directory, which can then be referenced by Xcode or copied into the BakBeat app bundle.

---

## Overview

This is the BakBeat meta-fork of libgpod, based on the gerion0/libgpod fork which modernized the build system to Meson. Key characteristics:

- **Purpose**: Provide a stable C library for parsing and writing iTunesDB/iTunesCDB from classic iPods
- **Build system**: Meson (not autotools)
- **Target**: macOS (Sonoma+), though the library is cross-platform
- **Output**: Dynamic library (`libgpod.dylib`) and headers for use from Swift/ObjC via a bridge layer

---

## Prerequisites (Homebrew)

Install the following packages via Homebrew:

```bash
brew install glib libplist sqlite pkg-config
```

### Package details:

| Package | Purpose |
|---------|---------|
| `glib` | Core GLib library (data structures, utilities) |
| `libplist` | Apple property list parsing (required for device info) |
| `sqlite` | SQLite database support (used by newer iPod models) |
| `pkg-config` | Build configuration tool |

### Optional packages:

```bash
# Only needed if you want to build Python bindings (not required for BakBeat)
brew install swig

# Only needed if you want to build Mono/.NET bindings (not required for BakBeat)
brew install mono
```

---

## Meson Build Commands

From the root of the bakbeat-libgpod repository:

```bash
cd ~/Github/bakbeat-libgpod

# Clean any previous build (optional, but recommended for fresh builds)
rm -rf builddir

# Configure the build
meson setup builddir \
  --prefix=$PWD/_install \
  -Dpython=disabled \
  -Dtest=false

# Compile
meson compile -C builddir

# Install to _install/
meson install -C builddir
```

### Build options explained:

| Option | Purpose |
|--------|---------|
| `--prefix=$PWD/_install` | Install artifacts to a local `_install/` directory rather than system paths |
| `-Dpython=disabled` | Skip Python bindings (not needed for BakBeat, avoids swig/pygobject dependencies) |
| `-Dtest=false` | Skip building test binaries (faster build, avoids taglib dependency) |

### Successful build produces:

```
_install/
├── lib/
│   ├── libgpod.4.dylib      # Versioned dynamic library
│   ├── libgpod.dylib        # Symlink to versioned library
│   └── pkgconfig/
│       └── gpod.pc          # pkg-config file for build integration
├── include/
│   └── gpod-1.0/
│       └── gpod/
│           ├── itdb.h       # Main API header
│           └── ...          # Other headers
├── bin/
│   └── ipod-read-sysinfo-extended   # Utility tool
└── share/
    └── locale/              # Translations (optional)
```

---

## Verifying the Build

After a successful build, verify the artifacts:

### Check library files:

```bash
ls -la _install/lib/
```

Expected output should include:
```
libgpod.4.dylib
libgpod.dylib -> libgpod.4.dylib
pkgconfig/
```

### Check headers:

```bash
ls _install/include/gpod-1.0/gpod/
```

Expected output should include:
```
itdb.h
```

### Verify pkg-config:

```bash
# Option 1: Use the .pc file directly
pkg-config --cflags --libs ./_install/lib/pkgconfig/gpod.pc

# Option 2: Set PKG_CONFIG_PATH
PKG_CONFIG_PATH=_install/lib/pkgconfig pkg-config --cflags --libs gpod
```

A healthy output looks like:
```
-I/path/to/bakbeat-libgpod/_install/include/gpod-1.0 -L/path/to/bakbeat-libgpod/_install/lib -lgpod
```

This confirms:
- Header search path (`-I...gpod-1.0`)
- Library search path (`-L..._install/lib`)
- Library to link (`-lgpod`)

---

## Troubleshooting

### Missing libplist-2.0

If you see an error about `libplist-2.0` not found:

```bash
# Ensure libplist is installed
brew install libplist

# Check pkg-config can find it
pkg-config --modversion libplist-2.0
```

### Python binding errors (when not disabled)

If you accidentally try to build Python bindings without dependencies:

```bash
# Re-run setup with Python disabled
meson setup builddir --wipe -Dpython=disabled -Dtest=false
```

### ARM64 vs x86_64

The library builds for the native architecture. If you need a universal binary for distribution, you'll need to build twice and use `lipo` to combine them.

---

## Integration with BakBeat

After building, the artifacts in `_install/` can be:

1. **Copied** into `RetroPod/ThirdParty/libgpod/` for vendoring
2. **Referenced** via header/library search paths in Xcode
3. **Bundled** into the app's Frameworks directory for distribution

See the BakBeat repository's `Docs/LibGpodIntegration.md` for detailed integration instructions.

---

## Additional Resources

- [README.overview](../README.overview) - High-level libgpod architecture
- [README.SysInfo](../README.SysInfo) - Device SysInfo handling
- [README.sqlite](../README.sqlite) - SQLite database support for newer iPods

