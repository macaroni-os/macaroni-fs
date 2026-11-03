<p align="center">
  <img src="https://github.com/macaroni-os/macaroni-site/blob/master/site/static/images/logo.png">
</p>

# Macaroni from Scratch A.K.A. MFS

The MFS repository contains the `anise-build` specs to build stage1 and stage2
packages.

## Building locally

### Prerequisites

- `anise` and `app-admin/anise-build`;
- a working Docker daemon - the build backend runs each package in a
  container seeded from the spec's seed image. In alternative could
  be exported the `DOCKER_HOST=<node>:2376` environment variable to
  point to a remote Docker daemon.

### Validate the specs

```
make validate
```

This is the same gate the CI workflow runs.

### Build the packages

Pass `PACKAGES=` with one or more `<category>/<name>` package IDs:

```
make build PACKAGES=seed/builder
make build PACKAGES=step1/gcc
```

`step1/gcc` transitively builds the whole current stage1 DAG:
`seed/builder` -> `step1/base` -> `linux-headers` -> `binutils` -> `gmp` ->
`mpfr` -> `mpc` -> `gcc`.

Notes:

- Artifacts land in `./build` as `<name>-<category>-<version>.package.tar.zst`.
  The payload is the tree relative to the spec's `package_dir`, so the step1
  toolchain payloads unpack under `$MFS` (default `/mfs`).
- The specs build single-threaded (LFS-style determinism); the cross-GCC step
  alone takes ~35 minutes on a modern desktop CPU.

### Create and serve the repository

When the build has completed, generate the repository metadata over the
built packages:

```
make create-repo
```

At the moment this follows the Macaroni OS publishing workflow, which uses
Mottainai and/or S3 as the target storage; a local-only workflow is still
being designed.

To test the repository without pushing it to a remote server, serve `./build`
over HTTP (port 8000):

```
make serve-repo
```

### Current stage1 scope

The chain currently stops at the cross GCC: glibc and the pass-2 binutils/GCC
builds (and the whole `step2` tree) are not specified yet, so a green build
verifies the cross-toolchain only.
