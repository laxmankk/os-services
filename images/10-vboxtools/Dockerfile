FROM gcc:10.3.0 as build-essential

ENV KERNEL_VERSION 5.10.129-burmilla

ENV KERNEL_HEADERS https://github.com/burmilla/os-kernel/releases/download/v${KERNEL_VERSION}/linux-${KERNEL_VERSION}-x86.tar.gz
ENV KERNEL_BASE https://github.com/burmilla/os-kernel/releases/download/v${KERNEL_VERSION}/linux-${KERNEL_VERSION}-x86.tar.gz
ENV VBOX_VERSION 6.1.36
ENV VBOX_SHA256 c987cdc8c08c579f56d921c85269aeeac3faf636babd01d9461ce579c9362cdd

RUN set -x \
 && apt-get update \
 && apt-get install -y --no-install-recommends p7zip-full libelf-dev \
 && apt-get clean \
 && rm -rf /var/lib/apt/*

RUN set -x \
 && mkdir -p /usr/src/v${KERNEL_VERSION} \
 && curl -sfL ${KERNEL_BASE} | tar zxf - -C / \
 && curl -sfL ${KERNEL_HEADERS} | tar zxf - -C /usr/src/v${KERNEL_VERSION}

RUN set -x \
 && wget -nv -O /vbox.iso "https://download.virtualbox.org/virtualbox/$VBOX_VERSION/VBoxGuestAdditions_$VBOX_VERSION.iso" \
 && echo "${VBOX_SHA256} */vbox.iso" | sha256sum -c - \
 && 7z x -o/ /vbox.iso VBoxLinuxAdditions.run \
 && rm /vbox.iso \
 && sh /VBoxLinuxAdditions.run --noexec --target /usr/src/vbox \
 && mkdir /usr/src/vbox/amd64 \
 && 7z x -so /usr/src/vbox/VBoxGuestAdditions-amd64.tar.bz2 | tar --extract --directory /usr/src/vbox/amd64 \
 && rm /usr/src/vbox/VBoxGuestAdditions-*.tar.bz2 \
 && ln -sT "vboxguest-$VBOX_VERSION" /usr/src/vbox/amd64/src/vboxguest

RUN set -x \
 && make \
      -C /usr/src/vbox/amd64/src/vboxguest \
      -j "$(nproc)" \
      KERN_DIR="/lib/modules/${KERNEL_VERSION}/build" \
      KERN_VER="${KERNEL_VERSION}" \
      vboxguest vboxsf


FROM debian:stable-slim

WORKDIR /dist

RUN set -x \
 && apt-get update \
 && apt-get install -y --no-install-recommends kmod \
 && apt-get clean \
 && rm -rf /var/lib/apt/*

COPY run /usr/local/bin/
COPY --from=build-essential /usr/src/vbox/amd64/src/vboxguest/vboxguest.ko .
COPY --from=build-essential /usr/src/vbox/amd64/src/vboxguest/vboxsf.ko .
COPY --from=build-essential /usr/src/vbox/amd64/other/mount.vboxsf .
COPY --from=build-essential /usr/src/vbox/amd64/sbin/VBoxService /sbin/
COPY --from=build-essential /usr/src/vbox/amd64/bin/VBoxControl /bin/
RUN chmod +x /usr/local/bin/run

CMD ["/usr/local/bin/run"]

ENTRYPOINT ["/usr/bin/ros", "entrypoint"]
