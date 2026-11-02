<p align="center">
  <img src="https://github.com/macaroni-os/macaroni-site/blob/master/site/static/images/logo.png">
</p>

# Macaroni from Scratch A.K.A. MFS

The MFS repository contains the `anise-build` specs to build stage1 and stage2
packages.

## Building locally

### Prerequisites

- `anise` and `app-admin/anise-build` (tested with anise-build 0.45.1);
- a working Docker daemon - the `docker` backend builds each package in a
  container seeded from the spec's seed image.

### Validate the specs

```
make validate
```

This is the same gate the CI workflow runs.

### Build the packages

Pass `PACKAGES=` with one or more `<category>/<name>` package IDs:

```
make build PACKAGES=seed/builder BACKEND=docker REPO_CACHE=
make build PACKAGES=step1/gcc BACKEND=docker REPO_CACHE=
```

`step1/gcc` transitively builds the whole current stage1 DAG:
`seed/builder` -> `step1/base` -> `linux-headers` -> `binutils` -> `gmp` ->
`mpfr` -> `mpc` -> `gcc`.

Notes:

- `BACKEND=docker` overrides the current Makefile default (`dockerv3`), which
  released anise-build 0.45.1 rejects with `invalid backend dockerv3.
  Unsupported` (it accepts `docker` and `img`).
- `REPO_CACHE=` disables the shared build-image repository
  (`macaronios/mfs-anise-cache`), which only project members can push to;
  the local Docker image cache serves the same purpose.
- `make build` with an empty `PACKAGES` selects nothing and exits 0 with
  `No packages selected for build!`. `anise-build build --all` currently
  reports the same despite selecting the whole tree; until both are
  addressed, pass explicit package IDs (e.g. all step1 IDs at once).
- Artifacts land in `./build` as `<name>-<category>-<version>.package.tar.zst`.
  The payload is the tree relative to the spec's `package_dir`, so the step1
  toolchain payloads unpack under `$MFS` (default `/mfs`).
- The specs build single-threaded (LFS-style determinism); the cross-GCC step
  alone takes ~35 minutes on a modern desktop CPU.

### Current stage1 scope

The chain currently stops at the cross GCC: glibc and the pass-2 binutils/GCC
builds (and the whole `step2` tree) are not specified yet, so a green build
verifies the cross-toolchain only.
