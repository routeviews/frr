VERSION 0.6

ARG DEFAULT_DISTRO=ubuntu
ARG DEFAULT_RELEASE=22.04

all:
    BUILD +pkg-march --DISTRO=ubuntu --RELEASE=20.04 # focal
    BUILD +pkg-march --DISTRO=ubuntu --RELEASE=22.04 # jammy

    BUILD +pkg-march --DISTRO=centos --RELEASE=7
    BUILD +pkg-march --DISTRO=centos --RELEASE=8

os-one:
    ARG --required DISTRO
    ARG --required RELEASE
    ARG --required TARGETARCH
    FROM ${DISTRO}:${RELEASE}
    WORKDIR /build

    IF [ "${DISTRO}" = "centos" ]
        IF [ "${RELEASE}" = "8" ]
            RUN sed -i 's/^mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-Linux-* && \
                sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-Linux-*
            RUN dnf install -y dnf-plugins-core
            RUN dnf config-manager --set-enabled powertools
            RUN yum -y install epel-release
            RUN dnf install -y \
                ca-certificates
        ELSE
            RUN yum install -y --setopt=skip_missing_names_on_install=False \
                ca-certificates
        END
    ELSE
        RUN DEBIAN_FRONTEND=noninteractive \
            apt-get update && apt-get install -y --no-install-recommends \
            ca-certificates \
            wget
    END

deps-one:
    ARG --required DISTRO
    ARG --required RELEASE
    ARG --required TARGETARCH
    FROM +os-one --DISTRO=${DISTRO} --RELEASE=${RELEASE}

    IF [ "${DISTRO}" = "centos" ]
       RUN yum install -y --setopt=skip_missing_names_on_install=False \
           git cmake \
           autoconf automake \
           libtool make \
           readline-devel \
           texinfo \
           net-snmp-devel \
           groff \
           pkgconfig \
           json-c-devel \
           pam-devel \
           bison flex \
           c-ares-devel \
           libcap-devel \
           elfutils-libelf-devel \
           libssh libssh-devel \
           libunwind-devel
       IF [ "${RELEASE}" = "7" ]
          RUN yum install -y --setopt=skip_missing_names_on_install=False \
              pytest python-devel python-sphinx
       ELSE
          RUN yum install -y --setopt=skip_missing_names_on_install=False \
              python3-pytest python3-devel python3-sphinx
       END

       # libyang
       RUN git clone https://github.com/CESNET/libyang.git /libyang && \
           cd /libyang && \
           git checkout v2.0.0 && \
           mkdir build && cd build && \
           cmake -D CMAKE_INSTALL_PREFIX:PATH=/usr \
                 -D CMAKE_BUILD_TYPE:String="Release" .. && \
           make && make install

       # librtr
       # TODO

    ELSE
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
            wget
    END

    # only "new" releases have libyang and librtr
    IF [ "${TARGETARCH}" = "arm64" ]
        ARG frr_arch="arm8"
    ELSE
        ARG frr_arch="x86_64"
    END

    IF [ "${DISTRO}" = "centos" ]

    ELSE
    IF [ "$DISTRO" = "ubuntu" ] && [ "${RELEASE}" = "20.04" ]
        # libyang
        ARG yang_base="https://ci1.netdef.org/artifact/LIBYANG-LIBYANGV2/shared/build-12"
        ARG yang_pkg="${yang_base}/Ubuntu-${RELEASE}-${frr_arch}-Packages"
        RUN wget "${yang_pkg}/libyang2_2.0.7-1~${DISTRO}${RELEASE}u1_${TARGETARCH}.deb" && \
            wget "${yang_pkg}/libyang2-dev_2.0.7-1~${DISTRO}${RELEASE}u1_${TARGETARCH}.deb"
        # librtr
        ARG rtr_base="https://ci1.netdef.org/artifact/RPKI-RTRLIB/shared/build-00149"
        ARG rtr_pkg="${rtr_base}/Ubuntu-${RELEASE}-${frr_arch}-Packages"
        RUN wget "${rtr_pkg}/librtr-dev_0.8.0_${TARGETARCH}.deb" && \
            wget "${rtr_pkg}/librtr0_0.8.0_${TARGETARCH}.deb"
        ARG extra_pkgs="./*.deb"
    ELSE
        ARG extra_pkgs="libyang2-dev librtr-dev"
    END
    RUN DEBIAN_FRONTEND=noninteractive \
        apt-get install -y --no-install-recommends \
                ${extra_pkgs}

    COPY --dir debian ./
    RUN mk-build-deps --remove debian/control && \
        DEBIAN_FRONTEND=noninteractive \
        apt install -y --no-install-recommends \
            ./frr-build-deps_8.5~dev-1_all.deb
    END

pkg-one:
        ARG DISTRO=${DEFAULT_DISTRO}
        ARG RELEASE=${DEFAULT_RELEASE}
        ARG TARGETPLATFORM
        FROM +deps-one --DISTRO=${DISTRO} --RELEASE=${RELEASE}
        COPY --dir * .
        IF [ "${DISTRO}" = "centos" ]
           RUN ./bootstrap.sh && \
               ./configure && \
               make dist
        ELSE
           RUN dpkg-buildpackage -b -rfakeroot -us -uc
           SAVE ARTIFACT ../*.deb AS LOCAL ./dist/${TARGETPLATFORM}/${DISTRO}/${RELEASE}/
        END

pkg-march:
        ARG --required DISTRO
        ARG --required RELEASE
        BUILD \
              --platform=linux/arm64 \
              --platform=linux/amd64 \
              +pkg-one \
                  --DISTRO=${DISTRO} --RELEASE=${RELEASE}
