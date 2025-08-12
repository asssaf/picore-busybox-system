# syntax=docker/dockerfile-upstream:master
FROM debian:stable AS build

ARG ARCH

RUN apt-get update \
        && apt-get install -y build-essential crossbuild-essential-${ARCH} squashfs-tools curl \
        && apt-get clean

ENV CC=${ARCH/armhf/arm-linux-gnueabihf-}
ENV CC=${CC/arm64/aarch64-linux-gnu-}

ARG BUSYBOX_VERSION=1.36.1

RUN mkdir /work
WORKDIR /work
RUN curl -sLo - "https://busybox.net/downloads/busybox-${BUSYBOX_VERSION}.tar.bz2" | tar xvj

WORKDIR busybox-${BUSYBOX_VERSION}
RUN make allnoconfig

ENV ENABLE_FEATURES="LONG_OPTS SHOW_USAGE FEATURE_VERBOSE_USAGE FEATURE_COMPRESS_USAGE INSTALL_APPLET_HARDLINKS LFS INSTALL_NO_USR \
                SV SVC SVOK SVLOGD RUNSV RUNSVDIR CHPST WATCHDOG"

ENV DISABLE_FEATURES="INSTALL_APPLET_SYMLINKS"

RUN ( \
        for F in ${ENABLE_FEATURES}; \
        do \
                sed -i "s/^.*# CONFIG_${F} is not set.*$/CONFIG_${F}=y/" .config ; \
        done; \
)

RUN ( \
        for F in ${DISABLE_FEATURES}; \
        do \
                sed -i "s/^CONFIG_${F}=y$/# CONFIG_${F} is not set/" .config ; \
        done; \
)

RUN make oldconfig
RUN make CROSS_COMPILE=$CC CC="${CC}gcc -Os -pipe" CXX="${CC}g++ -Os -pipe -fno-exceptions -fno-rtti" CONFIG_PREFIX=extension/usr/local busybox install
RUN rm extension/usr/local/bin/busybox

ARG TCZ=busybox-system.tcz
RUN mksquashfs extension /${TCZ} -all-root -noappend

WORKDIR /
RUN unsquashfs -lc ${TCZ} | sed 's|^squashfs-root/||' > ${TCZ}.list
RUN md5sum ${TCZ} > ${TCZ}.md5.txt

FROM scratch AS export

ARG TCZ=busybox-system.tcz

COPY --from=build /${TCZ} /
COPY --from=build /${TCZ}.list /
COPY --from=build /${TCZ}.md5.txt /
