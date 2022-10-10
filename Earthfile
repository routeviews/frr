VERSION 0.6
FROM ubuntu:20.04
WORKDIR /build

all:
        BUILD +pkg

deps:
        RUN apt-get update && apt-get install -y wget

        # download deps we can't get from normal repos
        RUN wget https://ci1.netdef.org/artifact/RPKI-RTRLIB/shared/build-145/Ubuntu-20.04-arm8-Packages/librtr-dev_0.8.0_arm64.deb
        RUN wget https://ci1.netdef.org/artifact/RPKI-RTRLIB/shared/build-145/Ubuntu-20.04-arm8-Packages/librtr0_0.8.0_arm64.deb
        RUN wget 'https://ci1.netdef.org/artifact/LIBYANG-LIBYANGV2/shared/build-12/Ubuntu-20.04-arm8-Packages/libyang2_2.0.7-1~ubuntu20.04u1_arm64.deb'
        RUN wget 'https://ci1.netdef.org/artifact/LIBYANG-LIBYANGV2/shared/build-12/Ubuntu-20.04-arm8-Packages/libyang2-dev_2.0.7-1~ubuntu20.04u1_arm64.deb'

        RUN DEBIAN_FRONTEND=noninteractive \
            apt-get install -y --no-install-recommends \
                    autoconf automake \
                    libtool make \
                    build-essential \
                    devscripts equivs \
                    libreadline-dev texinfo \
                    pkg-config \
                    libpam0g-dev \
                    libjson-c-dev \
                    bison flex \
                    libc-ares-dev \
                    python3-dev python3-sphinx \
                    install-info \
                    libsnmp-dev \
                    perl \
                    libcap-dev \
                    libelf-dev \
                    libunwind-dev \
                    wget \
                    ./librtr0_0.8.0_arm64.deb \
                    ./librtr-dev_0.8.0_arm64.deb \
                    './libyang2_2.0.7-1~ubuntu20.04u1_arm64.deb' \
                    './libyang2-dev_2.0.7-1~ubuntu20.04u1_arm64.deb'
        COPY --dir debian ./
        RUN mk-build-deps --remove debian/control && \
            DEBIAN_FRONTEND=noninteractive \
            apt install -y --no-install-recommends \
            ./frr-build-deps_8.5~dev-1_all.deb

pkg-deb:
        FROM +deps
        COPY --dir * .
        RUN dpkg-buildpackage -b -rfakeroot -us -uc
        ARG TARGETPLATFORM
        SAVE ARTIFACT ../*.deb AS LOCAL ./dist/

pkg:
        BUILD +pkg-deb
