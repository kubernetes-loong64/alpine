ARG ALPINE_VERSION=3.21.0
ARG MINIROOTFS_TARBALL=alpine-minirootfs.tar.gz

FROM scratch

ARG ALPINE_VERSION
ARG MINIROOTFS_TARBALL

LABEL maintainer="徐晓伟 <xuxiaowei@xuxiaowei.com.cn>" \
      license="Apache-2.0" \
      org.opencontainers.image.authors="徐晓伟 <xuxiaowei@xuxiaowei.com.cn>" \
      org.opencontainers.image.licenses="Apache-2.0" \
      org.opencontainers.image.title="Alpine Linux" \
      org.opencontainers.image.description="Alpine Linux ${ALPINE_VERSION} rootfs (multi-arch)" \
      org.opencontainers.image.version="${ALPINE_VERSION}" \
      org.opencontainers.image.url="https://github.com/kubernetes-loong64/alpine" \
      org.opencontainers.image.source="https://github.com/kubernetes-loong64/alpine"

ADD ${MINIROOTFS_TARBALL} /

CMD ["/bin/sh"]
