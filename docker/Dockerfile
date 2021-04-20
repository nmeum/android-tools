ARG ALPINE_VERSION=3.13

FROM alpine:${ALPINE_VERSION} as build_stage

ARG ANDROID_TOOLS_VERSION=31.0.0p1

RUN set -ex \
 && apk add \
            --quiet \
            --clean-protected \
            --no-cache \
            --update \
            --virtual .build-base \
              curl \
              build-base

RUN set -ex \
 && apk add \
            --quiet \
            --clean-protected \
            --no-cache \
            --update \
            --virtual .build-deps \
              pcre2-dev \
              linux-headers \
              libusb-dev \
              gtest-dev \
              go \
              perl \
              cmake \
              protobuf-dev \
              brotli-dev \
              zstd-dev \
              lz4-dev

RUN set -ex \
 && curl \
          -SLsf https://github.com/nmeum/android-tools/releases/download/${ANDROID_TOOLS_VERSION}/android-tools-${ANDROID_TOOLS_VERSION}.tar.xz \
          -o android-tools.tar.xz

RUN set -ex \
 && tar Jxf android-tools.tar.xz \
 && cd android-tools-${ANDROID_TOOLS_VERSION} \
 && cmake -B build \
          -DCMAKE_INSTALL_PREFIX=/usr \
          -DCMAKE_INSTALL_LIBDIR=lib \
          -DCMAKE_BUILD_TYPE=None \
  && cmake --build build \
  && DESTDIR="/install" cmake --install build

FROM alpine:${ALPINE_VERSION} as final_stage

RUN set -ex \
 && apk add \
            --quiet \
            --clean-protected \
            --no-cache \
            --update \
              libusb \
              libprotobuf \
              brotli-libs \
              zstd-libs \
              lz4-libs

COPY --from=build_stage /install /

COPY --chown=root:root rootfs/ /

ENTRYPOINT ["/usr/bin/adb"]

CMD ["-a", "-P", "5037", "server", "nodaemon"]

EXPOSE 5037

LABEL description="Provides alpine-based image with android platform tools."
