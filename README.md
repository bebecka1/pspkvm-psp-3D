# PSPKVM PSP first 3D build

This folder is a CI-ready build harness based on PSPKVM REDUX. It builds a PSP `EBOOT.PBP` with the existing **JSR-184/M3G** path enabled (m3gcore + pspgl + jsr184 component).

## Why this is the first build

- The repository you asked about (J2ME-Loader) cannot be compiled for PSP directly because it is an Android/Dalvik-ART converter app.
- PSPKVM REDUX already has a working PSP runtime + JSR-184 3D integration path.
- Mascot Capsule v3 / Deep 3D (SE) needs additional work; this build gives a real PSP 3D baseline first.

## Build locally (Docker)

```sh
git clone <this-repo>
cd pspkvm-psp-first3d

git clone --filter=blob:none https://github.com/vadosnaprimer/pspkvm.git src
git -C src checkout 15b93ccb82048d4ae12510ef65666bc13c79c252
git clone --filter=blob:none https://github.com/toaarnio/m3gcore.git m3gcore
git -C m3gcore checkout 1b921b3ae476b27d7359083babcbfab81d6e532f
git clone --filter=blob:none https://github.com/pspdev/pspgl.git pspgl
git -C pspgl checkout de4260adf56d06516ec46018d404ca77e0b61748

docker build -t pspkvm-build ./docker
docker run --rm \
  -v "$PWD/src:/work/pspkvm" \
  -v "$PWD/docker/patches:/work/patches:ro" \
  -v "$PWD/docker/patches-pspgl:/work/patches-pspgl:ro" \
  -v "$PWD/jsr184:/work/components/jsr184:ro" \
  -v "$PWD/m3g:/work/components/m3g:ro" \
  -v "$PWD/amr:/work/components/amr:ro" \
  -v "$PWD/m3gcore:/work/m3gcore:ro" \
  -v "$PWD/pspgl:/work/pspgl:ro" \
  pspkvm-build
```

Output: `src/psp/EBOOT.PBP`.

## Build in GitHub Actions

Push this folder as a GitHub repository. The workflow `.github/workflows/build.yml` builds the EBOOT and uploads artifacts:

- `EBOOT.PBP`
- `PSPKVM-PSP-first3d.zip` containing `PSP/GAME/PSPKVM/EBOOT.PBP`

## Install on PSP / PPSSPP

1. Download `PSPKVM-PSP-first3d.zip` artifact.
2. Extract `PSP/GAME/PSPKVM` to your Memory Stick root (`ms0:/PSP/GAME/PSPKVM`).
3. Launch **PSPKVM** from XMB/Game.
4. Install a JAR via PSPKVM menu.
5. For 3D games, select a 240x320 profile before launching.

## Test focus

- JSR-184/M3G games first (this build already includes m3gcore).
- Mascot Capsule v3 games (e.g., Deep 3D SE) are **not expected to be fully compatible yet**; they are the next integration target.

## Next step for Deep 3D / Mascot Capsule v3

Add a `mascotcapsule` component:
- Java: `com.mascotcapsule.micro3d.v3.*`
- KNI: `micro3d_render_kni.c`, loader/object natives
- Native PSP renderer: sceGu/VFPU, command list/figure/texture/effect
- Alias packages: `com.motorola.graphics.j3d`, `com.jblend.graphics.j3d`, etc.
