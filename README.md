# KiCad Docker Images (Debian Trixie)

Docker images for KiCad 9.0.5 built on Debian Trixie (13) with OpenCascade 7.8.1.

## Why This Repo?

Official KiCad Docker images use Debian Bookworm with OpenCascade 7.6.3, which has a [known bug](https://tracker.dev.opencascade.org/view.php?id=32977) that prevents proper color/material export in GLB files. This results in greyed-out 3D models.

This repo builds KiCad 9.0.5 on Debian Trixie (13) which includes OpenCascade 7.8.1 with the fix.

## Usage

```bash
# Pull from GitHub Container Registry
docker pull ghcr.io/diodeinc/kicad:9.0.5-trixie-full

# Export GLB with proper colors
docker run --rm \
  -v $(pwd):/workspace \
  ghcr.io/diodeinc/kicad:9.0.5-trixie-full \
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

## Available Tags

- `ghcr.io/diodeinc/kicad:9.0.5-trixie-full` - KiCad 9.0.5 with 3D models on Debian Trixie

## The Bug

### Problem
KiCad GLB exports from official Docker containers (`kicad/kicad:9.0.3-full`) show greyed-out models with no component colors.

### Root Cause
OpenCascade 7.6.3 (in Debian Bookworm) has a bug reading STEP material/color attributes. Fixed in OCCT 7.7.1.

### Evidence
- Official images (Bookworm/OCCT 7.6.3): 4 materials (PCB substrate only)
- This image (Trixie/OCCT 7.8.1): 17-20 materials (full color)

### Comparison

| Image | Debian | OCCT | Materials | Colors |
|-------|--------|------|-----------|--------|
| `kicad/kicad:9.0.5-full` | Bookworm | 7.6.3 | 4 | ❌ Grey |
| `ghcr.io/diodeinc/kicad:9.0.5-trixie-full` | Trixie | 7.8.1 | 20 | ✅ Full color |

## Building Locally

```bash
docker build \
  --build-arg include_3d=true \
  --build-arg KICAD_VERSION=9.0.5 \
  -t kicad:9.0.5-trixie-full \
  .
```

Build time: ~20-30 minutes

## License

This Dockerfile is based on the official KiCad Docker images from https://gitlab.com/kicad/packaging/kicad-cli-docker

KiCad is licensed under GPL-3.0-or-later.
