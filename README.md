# KiCad Docker Images with OCCT 7.8

Docker images for KiCad with OpenCascade 7.8.1 for proper GLB color export.

## Why This Repo?

Official KiCad Docker images use Debian Bookworm with OpenCascade 7.6.3, which has a [known bug](https://tracker.dev.opencascade.org/view.php?id=32977) that prevents proper color/material export in GLB files. This results in greyed-out 3D models.

These images use newer base systems with OpenCascade 7.8.1 which fixes the color export issue.

## Available Images

### Ubuntu 25.10
- **Tag:** `ghcr.io/dioderobot/kicad:9.0.3-ubuntu-25.10`
- **KiCad:** 9.0.3 from Ubuntu packages
- **Size:** ~7GB
- **Build time:** ~2 minutes

### Debian Trixie (Recommended)
- **Tag:** `ghcr.io/dioderobot/kicad:9.0.6-trixie-full`
- **KiCad:** 9.0.6 compiled from source
- **Size:** ~3GB
- **Build time:** ~30 minutes

## Usage

```bash
# Pull Ubuntu image (recommended)
docker pull ghcr.io/dioderobot/kicad:9.0.3-ubuntu-25.10

# Export GLB with proper colors
docker run --rm \
  -v $(pwd):/workspace \
  ghcr.io/dioderobot/kicad:9.0.3-ubuntu-25.10 \
  kicad-cli pcb export glb \
  --output /workspace/output.glb \
  --subst-models \
  --force \
  --no-dnp \
  --no-unspecified \
  --include-pads \
  --include-silkscreen \
  --include-soldermask \
  --include-tracks \
  --include-zones \
  /workspace/your-board.kicad_pcb
```

## The Bug

### Problem
KiCad GLB exports from official Docker containers (`kicad/kicad:9.0.6-full`) show greyed-out models with no component colors.

### Root Cause
OpenCascade 7.6.3 (in Debian Bookworm) has a bug reading STEP material/color attributes. Fixed in OCCT 7.7.1.

### Evidence
- Official images (Bookworm/OCCT 7.6.3): 4 materials (PCB substrate only)
- This image (Trixie/OCCT 7.8.1): 17-20 materials (full color)

### Comparison

| Image | Base OS | OCCT | Materials | Colors |
|-------|---------|------|-----------|--------|
| `kicad/kicad:9.0.6-full` | Debian Bookworm | 7.6.3 | 4 | ❌ Grey |
| `ghcr.io/dioderobot/kicad:9.0.3-ubuntu-25.10` | Ubuntu 25.10 | 7.8.1 | 19 | ✅ Full color |
| `ghcr.io/dioderobot/kicad:9.0.6-trixie-full` | Debian Trixie | 7.8.1 | 20 | ✅ Full color |

## Building Locally

### Ubuntu (fast)
```bash
docker build -f Dockerfile.ubuntu -t kicad:ubuntu .
```

### Debian Trixie (latest KiCad)
```bash
docker build --build-arg include_3d=true --build-arg KICAD_VERSION=9.0.6 -t kicad:trixie .
```

## License

This Dockerfile is based on the official KiCad Docker images from https://gitlab.com/kicad/packaging/kicad-cli-docker

KiCad is licensed under GPL-3.0-or-later.
