# Alpine Linux for LoongArch64

<p align="center"><a href="README.md">English</a> | <a href="README-zh.md">中文</a></p>

<p align="center"><img src="https://img.shields.io/badge/Alpine%20Linux%20LoongArch64%20%E9%BE%99%E8%8A%AF%E6%9E%B6%E6%9E%84%E5%8F%91%E8%A1%8C%E7%89%88-blue?logo=alpinelinux&logoColor=white" alt="Alpine Linux LoongArch64 龙芯架构发行版"></p>

Build [Alpine Linux](https://alpinelinux.org/) base container images for the **LoongArch64 (loong64)** architecture —
and eight other architectures — via CI/CD, and publish them as multi-architecture images on Docker Hub together with
GPG-signed image tarballs on GitHub Releases.

Images are assembled from the official upstream Alpine minirootfs tarballs, so each image contains the unmodified
Alpine userland in a single `scratch`-based layer — no extra packages, layers or patches.

## How it works

A GitHub Actions workflow, published on the `loong64-*` work branches, does the following for every entry of the
build matrix:

1. Resolves the Alpine version from the branch or tag name (e.g. `loong64-3.24.2` → `3.24.2`).
2. Downloads `alpine-minirootfs-<version>-<alpine-arch>.tar.gz` and its `.sha256` from the upstream mirror and
   verifies the checksum. The exact patch version is preferred; when upstream does not publish a minirootfs for that
   patch release, the newest minirootfs of the same minor branch (e.g. `v3.24`) is used instead.
3. Imports the tarball into a `scratch` image using `docker/Dockerfile`, producing
   `kubernetesloong64/alpine:<version>-<alpine-arch>`.
4. Runs the image and prints `/etc/os-release` to verify the runtime.
5. Saves the image as `alpine-<version>-<alpine-arch>.tar` for the release.

On a tag build the per-architecture images are pushed to Docker Hub, combined into a multi-architecture manifest list
(`kubernetesloong64/alpine:<version>`), signed with GPG and published as a GitHub Release.

## Branch naming

Push a branch named `loong64-<alpine-version>` (e.g. `loong64-3.24.2`) to trigger a build.
Append `+<build>` (e.g. `loong64-3.24.2+0`) to include build metadata.

One branch exists per Alpine patch release, grouped by Alpine minor version line:

| Alpine version line | Branches                                 |
|---------------------|------------------------------------------|
| 3.21                | `loong64-3.21.0` … `loong64-3.21.8`      |
| 3.22                | `loong64-3.22.0` … `loong64-3.22.6`      |
| 3.23                | `loong64-3.23.0` … `loong64-3.23.6`      |
| 3.24                | `loong64-3.24.0` … `loong64-3.24.2`      |

## [Release](https://github.com/kubernetes-loong64/alpine/releases)

Push a tag matching `release-loong64-<alpine-version>` (e.g. `release-loong64-3.24.2`) to publish a GitHub Release
with the built Docker images.

The `+<build>` suffix provides build metadata (e.g. `+0`, `+1-alpha.1`). When build metadata is present, the images
are published under both the full tag and the plain version tag — for example `3.24.2-1-alpha.1` and `3.24.2` for the
branch `loong64-3.24.2+1-alpha.1`.

The suffix in the build metadata indicates the release stage:

| Suffix  | Stage         |
|---------|---------------|
| `alpha` | Internal beta |
| `beta`  | Public beta   |
| `rc`    | Pre-release   |
| (none)  | Stable        |

## Supported architectures

| Alpine arch (minirootfs) | Docker platform   | Image tag suffix |
|--------------------------|-------------------|------------------|
| `x86_64`                 | `linux/amd64`     | `-x86_64`        |
| `aarch64`                | `linux/arm64`     | `-aarch64`       |
| `armv7`                  | `linux/arm/v7`    | `-armv7`         |
| `armhf`                  | `linux/arm/v6`    | `-armhf`         |
| `x86`                    | `linux/386`       | `-x86`           |
| `loongarch64`            | `linux/loong64`   | `-loongarch64`   |
| `ppc64le`                | `linux/ppc64le`   | `-ppc64le`       |
| `s390x`                  | `linux/s390x`     | `-s390x`         |
| `riscv64`                | `linux/riscv64`   | `-riscv64`       |

The LoongArch target platform is `linux/loong64`, which corresponds to the upstream Alpine architecture directory
`loongarch64`.

## Docker images

Images are published to Docker Hub under
[`kubernetesloong64/alpine`](https://hub.docker.com/r/kubernetesloong64/alpine).

- [![kubernetesloong64/alpine:3.21](https://img.shields.io/badge/kubernetesloong64%2Falpine-3.21-blue?labelColor=gray&logo=docker)](https://hub.docker.com/r/kubernetesloong64/alpine/tags?name=3.21)
- [![kubernetesloong64/alpine:3.22](https://img.shields.io/badge/kubernetesloong64%2Falpine-3.22-blue?labelColor=gray&logo=docker)](https://hub.docker.com/r/kubernetesloong64/alpine/tags?name=3.22)
- [![kubernetesloong64/alpine:3.23](https://img.shields.io/badge/kubernetesloong64%2Falpine-3.23-blue?labelColor=gray&logo=docker)](https://hub.docker.com/r/kubernetesloong64/alpine/tags?name=3.23)
- [![kubernetesloong64/alpine:3.24](https://img.shields.io/badge/kubernetesloong64%2Falpine-3.24-blue?labelColor=gray&logo=docker)](https://hub.docker.com/r/kubernetesloong64/alpine/tags?name=3.24)
- [![kubernetesloong64/alpine:loongarch64](https://img.shields.io/badge/kubernetesloong64%2Falpine-loongarch64-blue?labelColor=gray&logo=docker)](https://hub.docker.com/r/kubernetesloong64/alpine/tags?name=loongarch64)

| Registry                         | Image                                                                  |
|----------------------------------|------------------------------------------------------------------------|
| Docker Hub                       | `kubernetesloong64/alpine:<tag>`                                       |
| Alibaba Cloud (Qingdao) mirror   | `registry.cn-qingdao.aliyuncs.com/kubernetesloong64/alpine:<tag>`      |

The mirror is populated only when the optional `SYNC_REGISTRY` repository variable is configured.

### Available tags

| Tag                | Description                                                          |
|--------------------|----------------------------------------------------------------------|
| `<version>`        | Multi-architecture manifest list covering all nine architectures     |
| `<version>-<arch>` | Single-architecture image, e.g. `3.24.2-loongarch64` for LoongArch64 |

### Pull images

```shell
# Multi-architecture manifest list
docker pull kubernetesloong64/alpine:3.24.2

# LoongArch64 (loong64) only
docker pull kubernetesloong64/alpine:3.24.2-loongarch64

# Run on LoongArch64
docker run --rm --platform linux/loong64 kubernetesloong64/alpine:3.24.2 cat /etc/os-release
```

### Use as a base image

```dockerfile
FROM kubernetesloong64/alpine:3.24.2-loongarch64

RUN apk add --no-cache ca-certificates
```

## Release artifacts

Each release includes one image tarball per architecture, and every tarball has a corresponding `.asc` detached GPG
signature:

| File                               | Architecture |
|------------------------------------|--------------|
| `alpine-<version>-x86_64.tar`      | amd64        |
| `alpine-<version>-aarch64.tar`     | arm64        |
| `alpine-<version>-armv7.tar`       | arm/v7       |
| `alpine-<version>-armhf.tar`       | arm/v6       |
| `alpine-<version>-x86.tar`         | 386          |
| `alpine-<version>-loongarch64.tar` | loong64      |
| `alpine-<version>-ppc64le.tar`     | ppc64le      |
| `alpine-<version>-s390x.tar`       | s390x        |
| `alpine-<version>-riscv64.tar`     | riscv64      |

Download and load an image:

```shell
curl -LO https://github.com/kubernetes-loong64/alpine/releases/download/release-loong64-3.24.2/alpine-3.24.2-loongarch64.tar
docker load -i alpine-3.24.2-loongarch64.tar
```

## Building locally

The build pipeline and `docker/Dockerfile` live on the `loong64-<version>` work branches, so switch to the branch for
the Alpine version you want to build first:

```shell
git switch loong64-3.24.2
cd docker
curl -fLO https://dl-cdn.alpinelinux.org/alpine/v3.24/releases/loongarch64/alpine-minirootfs-3.24.2-loongarch64.tar.gz
docker build \
  --build-arg ALPINE_VERSION=3.24.2 \
  --build-arg MINIROOTFS_TARBALL=alpine-minirootfs-3.24.2-loongarch64.tar.gz \
  -t alpine:3.24.2-loongarch64 .
```

## Configuration

The workflow can be pointed at other registries and mirrors through repository variables and secrets:

| Name                     | Type     | Default                                 | Description                                           |
|--------------------------|----------|-----------------------------------------|-------------------------------------------------------|
| `NAMESPACE`              | variable | `kubernetesloong64`                     | Docker Hub namespace of the published images          |
| `ALPINE_MIRROR`          | variable | `https://dl-cdn.alpinelinux.org/alpine` | Upstream Alpine mirror serving the minirootfs         |
| `SYNC_REGISTRY`          | variable | —                                       | Optional second registry to mirror images to          |
| `SYNC_REGISTRY_USERNAME` | variable | —                                       | Username for the sync registry                        |
| `DOCKER_USERNAME`        | variable | —                                       | Docker Hub username                                   |
| `SYNC_REGISTRY_PASSWORD` | secret   | —                                       | Password for the sync registry                        |
| `DOCKER_PASSWORD`        | secret   | —                                       | Docker Hub password or access token                   |
| `GPG_PRIVATE_KEY_BASE64` | secret   | —                                       | Base64-encoded GPG private key used to sign artifacts |

## Verifying releases

- Releases are signed with GPG.
- Download the public key from [keys.openpgp.org](https://keys.openpgp.org).
- Fingerprint: [FCF8724722CCBF9F51B1FBE376532BE7E3013105](https://keys.openpgp.org/debug?q=FCF8724722CCBF9F51B1FBE376532BE7E3013105)
- [Manual download](https://keys.openpgp.org/vks/v1/by-fingerprint/FCF8724722CCBF9F51B1FBE376532BE7E3013105)

```shell
gpg --keyserver keys.openpgp.org --recv-keys FCF8724722CCBF9F51B1FBE376532BE7E3013105
echo "FCF8724722CCBF9F51B1FBE376532BE7E3013105:6:" | gpg --import-ownertrust
```

Or download the key file manually and import it:

```shell
gpg --import /tmp/xxx
```

Then verify a downloaded artifact against its detached signature:

```shell
gpg --verify alpine-3.24.2-loongarch64.tar.asc alpine-3.24.2-loongarch64.tar
```

## Related repositories

| Repository                                                                           | Description                                                               |
|--------------------------------------------------------------------------------------|---------------------------------------------------------------------------|
| [submodule-loong64](https://github.com/kubernetes-loong64/submodule-loong64)         | Monorepo aggregating all LoongArch64 component repositories as submodules |
| [debian-loong64](https://github.com/kubernetes-loong64/debian-loong64)               | Debian base images for LoongArch64                                        |
| [.github](https://github.com/kubernetes-loong64/.github/blob/main/profile/README.md) | Overview of every project in the kubernetes-loong64 organization          |

## License

[Apache License 2.0](LICENSE)
