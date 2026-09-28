# Alpine Linux for LoongArch64

<p align="center"><a href="README.md">English</a> | <a href="README-zh.md">中文</a></p>

<p align="center"><img src="https://img.shields.io/badge/Alpine%20Linux%20LoongArch64%20%E9%BE%99%E8%8A%AF%E6%9E%B6%E6%9E%84%E5%8F%91%E8%A1%8C%E7%89%88-blue?logo=alpinelinux&logoColor=white" alt="Alpine Linux LoongArch64 龙芯架构发行版"></p>

通过 CI/CD 为 **LoongArch64 (loong64)** 架构（以及其他八种架构）构建
[Alpine Linux](https://alpinelinux.org/) 基础容器镜像，并以多架构镜像的形式发布到 Docker Hub，
同时在 GitHub Releases 中提供 GPG 签名的镜像包。

镜像使用 Alpine 官方的 minirootfs 压缩包组装，因此每个镜像仅包含一个基于 `scratch` 的层，
其中的 Alpine 用户空间未经任何修改——没有额外的软件包、层或补丁。

## 工作原理

发布在 `loong64-*` 工作分支上的 GitHub Actions 工作流会针对构建矩阵中的每一项执行以下操作：

1. 从分支名或标签名解析 Alpine 版本（例如 `loong64-3.24.2` → `3.24.2`）。
2. 从上游镜像源下载 `alpine-minirootfs-<version>-<alpine-arch>.tar.gz` 及其 `.sha256` 文件并校验。
   优先使用精确的补丁版本；当上游未发布该补丁版本的 minirootfs 时，改用同一小版本分支
   （例如 `v3.24`）中最新的 minirootfs。
3. 使用 `docker/Dockerfile` 将压缩包导入 `scratch` 镜像，生成
   `kubernetesloong64/alpine:<version>-<alpine-arch>`。
4. 运行该镜像并打印 `/etc/os-release` 以验证运行时。
5. 将镜像保存为 `alpine-<version>-<alpine-arch>.tar` 用于发布。

在标签构建中，各架构镜像会被推送到 Docker Hub，合并为多架构清单列表
（`kubernetesloong64/alpine:<version>`），使用 GPG 签名，并发布为 GitHub Release。

## 分支命名

推送名为 `loong64-<alpine-version>` 的分支（例如 `loong64-3.24.2`）以触发构建。
可附加 `+<build>`（例如 `loong64-3.24.2+0`）来包含构建元数据。

每个 Alpine 补丁版本对应一个分支，并按 Alpine 小版本线分组：

| Alpine 版本线 | 分支                                |
|---------------|-------------------------------------|
| 3.21          | `loong64-3.21.0` … `loong64-3.21.8` |
| 3.22          | `loong64-3.22.0` … `loong64-3.22.6` |
| 3.23          | `loong64-3.23.0` … `loong64-3.23.6` |
| 3.24          | `loong64-3.24.0` … `loong64-3.24.2` |

## [发布](https://github.com/kubernetes-loong64/alpine/releases)

推送匹配 `release-loong64-<alpine-version>` 的标签（例如 `release-loong64-3.24.2`）即可发布 GitHub Release，
其中包含构建好的 Docker 镜像。

`+<build>` 后缀提供构建元数据（例如 `+0`、`+1-alpha.1`）。存在构建元数据时，镜像会同时以完整标签和
纯版本标签发布——例如分支 `loong64-3.24.2+1-alpha.1` 会发布 `3.24.2-1-alpha.1` 和 `3.24.2` 两个标签。

构建元数据中的后缀表示发布阶段：

| 后缀    | 阶段       |
|---------|------------|
| `alpha` | 内部测试版 |
| `beta`  | 公开测试版 |
| `rc`    | 预发布     |
| (无)    | 稳定版     |

## 支持的架构

| Alpine 架构（minirootfs） | Docker 平台     | 镜像标签后缀   |
|---------------------------|-----------------|----------------|
| `x86_64`                  | `linux/amd64`   | `-x86_64`      |
| `aarch64`                 | `linux/arm64`   | `-aarch64`     |
| `armv7`                   | `linux/arm/v7`  | `-armv7`       |
| `armhf`                   | `linux/arm/v6`  | `-armhf`       |
| `x86`                     | `linux/386`     | `-x86`         |
| `loongarch64`             | `linux/loong64` | `-loongarch64` |
| `ppc64le`                 | `linux/ppc64le` | `-ppc64le`     |
| `s390x`                   | `linux/s390x`   | `-s390x`       |
| `riscv64`                 | `linux/riscv64` | `-riscv64`     |

LoongArch 的目标平台为 `linux/loong64`，对应上游 Alpine 的架构目录 `loongarch64`。

## Docker 镜像

镜像发布在 Docker Hub 的
[`kubernetesloong64/alpine`](https://hub.docker.com/r/kubernetesloong64/alpine)。

- [![kubernetesloong64/alpine:3.21](https://img.shields.io/badge/kubernetesloong64%2Falpine-3.21-blue?labelColor=gray&logo=docker)](https://hub.docker.com/r/kubernetesloong64/alpine/tags?name=3.21)
- [![kubernetesloong64/alpine:3.22](https://img.shields.io/badge/kubernetesloong64%2Falpine-3.22-blue?labelColor=gray&logo=docker)](https://hub.docker.com/r/kubernetesloong64/alpine/tags?name=3.22)
- [![kubernetesloong64/alpine:3.23](https://img.shields.io/badge/kubernetesloong64%2Falpine-3.23-blue?labelColor=gray&logo=docker)](https://hub.docker.com/r/kubernetesloong64/alpine/tags?name=3.23)
- [![kubernetesloong64/alpine:3.24](https://img.shields.io/badge/kubernetesloong64%2Falpine-3.24-blue?labelColor=gray&logo=docker)](https://hub.docker.com/r/kubernetesloong64/alpine/tags?name=3.24)
- [![kubernetesloong64/alpine:loongarch64](https://img.shields.io/badge/kubernetesloong64%2Falpine-loongarch64-blue?labelColor=gray&logo=docker)](https://hub.docker.com/r/kubernetesloong64/alpine/tags?name=loongarch64)

| 镜像仓库           | 镜像                                                              |
|--------------------|-------------------------------------------------------------------|
| Docker Hub         | `kubernetesloong64/alpine:<tag>`                                  |
| 阿里云（青岛）镜像 | `registry.cn-qingdao.aliyuncs.com/kubernetesloong64/alpine:<tag>` |

仅当配置了可选的 `SYNC_REGISTRY` 仓库变量时，才会同步到镜像仓库。

### 可用标签

| 标签               | 描述                                                 |
|--------------------|------------------------------------------------------|
| `<version>`        | 覆盖全部九种架构的多架构清单列表                     |
| `<version>-<arch>` | 单架构镜像，例如 LoongArch64 的 `3.24.2-loongarch64` |

### 拉取镜像

```shell
# 多架构清单列表
docker pull kubernetesloong64/alpine:3.24.2

# 仅 LoongArch64 (loong64)
docker pull kubernetesloong64/alpine:3.24.2-loongarch64

# 在 LoongArch64 上运行
docker run --rm --platform linux/loong64 kubernetesloong64/alpine:3.24.2 cat /etc/os-release
```

### 作为基础镜像使用

```dockerfile
FROM kubernetesloong64/alpine:3.24.2-loongarch64

RUN apk add --no-cache ca-certificates
```

## 发布产物

每个发布包含每种架构一个镜像包，且每个包都有对应的 `.asc` 分离 GPG 签名：

| 文件                               | 架构    |
|------------------------------------|---------|
| `alpine-<version>-x86_64.tar`      | amd64   |
| `alpine-<version>-aarch64.tar`     | arm64   |
| `alpine-<version>-armv7.tar`       | arm/v7  |
| `alpine-<version>-armhf.tar`       | arm/v6  |
| `alpine-<version>-x86.tar`         | 386     |
| `alpine-<version>-loongarch64.tar` | loong64 |
| `alpine-<version>-ppc64le.tar`     | ppc64le |
| `alpine-<version>-s390x.tar`       | s390x   |
| `alpine-<version>-riscv64.tar`     | riscv64 |

下载并加载镜像：

```shell
curl -LO https://github.com/kubernetes-loong64/alpine/releases/download/release-loong64-3.24.2/alpine-3.24.2-loongarch64.tar
docker load -i alpine-3.24.2-loongarch64.tar
```

## 本地构建

构建流水线和 `docker/Dockerfile` 位于 `loong64-<version>` 工作分支上，因此请先切换到需要构建的
Alpine 版本所在分支：

```shell
git switch loong64-3.24.2
cd docker
curl -fLO https://dl-cdn.alpinelinux.org/alpine/v3.24/releases/loongarch64/alpine-minirootfs-3.24.2-loongarch64.tar.gz
docker build \
  --build-arg ALPINE_VERSION=3.24.2 \
  --build-arg MINIROOTFS_TARBALL=alpine-minirootfs-3.24.2-loongarch64.tar.gz \
  -t alpine:3.24.2-loongarch64 .
```

## 配置

可以通过仓库变量和密钥将工作流指向其他镜像仓库和镜像源：

| 名称                     | 类型 | 默认值                                  | 描述                                 |
|--------------------------|------|-----------------------------------------|--------------------------------------|
| `NAMESPACE`              | 变量 | `kubernetesloong64`                     | 发布镜像的 Docker Hub 命名空间       |
| `ALPINE_MIRROR`          | 变量 | `https://dl-cdn.alpinelinux.org/alpine` | 提供 minirootfs 的上游 Alpine 镜像源 |
| `SYNC_REGISTRY`          | 变量 | —                                       | 可选的第二镜像仓库，用于同步镜像     |
| `SYNC_REGISTRY_USERNAME` | 变量 | —                                       | 同步镜像仓库的用户名                 |
| `DOCKER_USERNAME`        | 变量 | —                                       | Docker Hub 用户名                    |
| `SYNC_REGISTRY_PASSWORD` | 密钥 | —                                       | 同步镜像仓库的密码                   |
| `DOCKER_PASSWORD`        | 密钥 | —                                       | Docker Hub 密码或访问令牌            |
| `GPG_PRIVATE_KEY_BASE64` | 密钥 | —                                       | 用于签名产物的 Base64 编码 GPG 私钥  |

## 验证发布

- 发布文件使用 GPG 签名。
- 从 [keys.openpgp.org](https://keys.openpgp.org) 下载公钥。
- 指纹：[FCF8724722CCBF9F51B1FBE376532BE7E3013105](https://keys.openpgp.org/debug?q=FCF8724722CCBF9F51B1FBE376532BE7E3013105)
- [手动下载](https://keys.openpgp.org/vks/v1/by-fingerprint/FCF8724722CCBF9F51B1FBE376532BE7E3013105)

```shell
gpg --keyserver keys.openpgp.org --recv-keys FCF8724722CCBF9F51B1FBE376532BE7E3013105
echo "FCF8724722CCBF9F51B1FBE376532BE7E3013105:6:" | gpg --import-ownertrust
```

或者，手动下载公钥文件后导入：

```shell
gpg --import /tmp/xxx
```

然后使用分离签名验证已下载的产物：

```shell
gpg --verify alpine-3.24.2-loongarch64.tar.asc alpine-3.24.2-loongarch64.tar
```

## 相关仓库

| 仓库                                                                                 | 描述                                                     |
|--------------------------------------------------------------------------------------|----------------------------------------------------------|
| [submodule-loong64](https://github.com/kubernetes-loong64/submodule-loong64)         | 以 git 子模块形式聚合所有 LoongArch64 组件仓库的单一仓库 |
| [debian-loong64](https://github.com/kubernetes-loong64/debian-loong64)               | 面向 LoongArch64 的 Debian 基础镜像                      |
| [.github](https://github.com/kubernetes-loong64/.github/blob/main/profile/README.md) | kubernetes-loong64 组织下所有项目的总览                  |

## 许可证

[Apache License 2.0](LICENSE)
