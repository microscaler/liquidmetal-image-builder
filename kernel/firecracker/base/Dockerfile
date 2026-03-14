# Default matches kernel/Makefile BUILDER_IMAGE_NAME (avoids invalid base image when built without --build-arg)
ARG IMAGE_NAME=ghcr.io/liquidmetal-dev/kernel-builder-amd64
FROM ${IMAGE_NAME}:dev AS builder

ENV SRC_DIR=/usr/src            \
    DIST_DIR=/dist              \
    LINUX_DIR=/usr/src/linux    \
    LINUX_REPO_URL=https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git

ARG KERNEL_VERSION
ARG CONFIG_ARCH
ARG KERNEL_ARCH
ARG CONFIG_FILE

RUN mkdir -p ${SRC_DIR} ${DIST_DIR} && \
    git clone --depth 1 --branch v${KERNEL_VERSION} ${LINUX_REPO_URL} ${LINUX_DIR} && \
    cd ${LINUX_DIR}

WORKDIR ${LINUX_DIR}

COPY firecracker/configs/${CONFIG_ARCH}/${CONFIG_FILE} .config

# Linux kernel Makefile expects ARCH=x86_64 or ARCH=arm64 (not amd64). Set it so make finds arch/$(ARCH)/Makefile.
ENV ARCH=${KERNEL_ARCH}

RUN make LOCALVERSION= olddefconfig
# -j32 tells the compiler to use 32 threads, if 32 threads are not available it will use as many as it can up to 32
RUN make LOCALVERSION= -j32
RUN make LOCALVERSION= modules_install

# x86_64: vmlinux -> /boot/vmlinux; arm64: make Image -> /boot/image
RUN if [ "$ARCH" = "x86_64" ]; then cp vmlinux /boot/vmlinux; else make Image && cp ./arch/arm64/boot/Image /boot/image; fi && \
    cp .config /boot/config-${KERNEL_VERSION}

FROM scratch
COPY --from=builder /boot /boot
COPY --from=builder /lib/modules /lib/modules
