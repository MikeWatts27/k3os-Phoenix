# K3OS kernel build for Ubuntu Jammy
ARG REPO
ARG TAG
FROM ubuntu:jammy

# Arguments
ARG ARCH
ENV KVERSION=5.4.0-88-generic
ENV URL=https://github.com/rancher/k3os-kernel/releases/download/5.4.0-88.99-rancher1
ENV KERNEL_XZ=${URL}/kernel-generic_${ARCH}.tar.xz
ENV KERNEL_EXTRA_XZ=${URL}/kernel-extra-generic_${ARCH}.tar.xz
ENV KERNEL_HEADERS_XZ=${URL}/kernel-headers-generic_${ARCH}.tar.xz

# Install required packages and configure modules
RUN apt-get update && \
    apt-get install -y curl initramfs-tools kmod lz4 rsync xz-utils cpio gzip && \
    echo -e "r8152\nhfs\nhfsplus\nnls_utf8\nnls_iso8859_1" >> /etc/initramfs-tools/modules && \
    rm -rf /var/lib/apt/lists/*

# Download kernel tarballs
RUN mkdir -p /usr/src && \
    curl -fL $KERNEL_XZ -o /usr/src/kernel.tar.xz && \
    curl -fL $KERNEL_EXTRA_XZ -o /usr/src/kernel-extra.tar.xz && \
    curl -fL $KERNEL_HEADERS_XZ -o /usr/src/kernel-headers.tar.xz

# Extract kernel, headers, and extra modules
RUN mkdir -p /usr/src/root && \
    tar -xf /usr/src/kernel.tar.xz -C /usr/src/root && \
    tar -xf /usr/src/kernel-extra.tar.xz -C /usr/src/root && \
    tar -xf /usr/src/kernel-headers.tar.xz -C /usr/src/root

# Create initrd targeting extracted kernel
RUN mkdir -p /usr/src/initrd && \
    rsync -a /usr/src/root/lib/ /usr/src/initrd/lib/ && \
    depmod -b /usr/src/root $KVERSION && \
    mkinitramfs -k $KVERSION -c lz4 -o /usr/src/initrd.tmp

# Generate initrd firmware and module lists
RUN mkdir -p /output/lib /output/headers && \
    cd /usr/src/initrd && \
    lz4cat /usr/src/initrd.tmp | cpio -idmv && \
    find lib/modules -name '*.ko' > /output/initrd-modules && \
    echo lib/modules/${KVERSION}/modules.order >> /output/initrd-modules && \
    echo lib/modules/${KVERSION}/modules.builtin >> /output/initrd-modules && \
    find lib/firmware -type f > /output/initrd-firmware && \
    find usr/lib/firmware -type f | sed 's!usr/!!' >> /output/initrd-firmware

# Copy final kernel artifacts
RUN cd /usr/src/root && \
    cp -r usr/src/linux-headers* /output/headers && \
    cp -r lib/firmware /output/lib/firmware && \
    cp -r lib/modules /output/lib/modules && \
    cp boot/System.map* /output/System.map && \
    cp boot/config* /output/config && \
    cp boot/vmlinuz-* /output/vmlinuz && \
    echo ${KVERSION} > /output/version
