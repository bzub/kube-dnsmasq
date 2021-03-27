FROM alpine:3.13 as git-clone
RUN apk --update --no-cache add git
RUN git clone --depth=1 git://git.ipxe.org/ipxe.git /ipxe
COPY version.patch /ipxe/version.patch
RUN git -C /ipxe apply /ipxe/version.patch

FROM alpine:3.13
RUN set -xe \
  && apk --update --no-cache add --virtual build-dependencies \
    binutils \
    gcc \
    make \
    musl \
    musl-dev \
    musl-utils \
    perl \
    syslinux \
    xorriso \
    xz-dev
COPY --from=git-clone /ipxe /usr/src/ipxe
WORKDIR /usr/src/ipxe/src
ENTRYPOINT /usr/bin/make
