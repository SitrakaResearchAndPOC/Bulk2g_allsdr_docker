# =========================================================
# Debian Buster + Systemd + Osmocom + SDR Stack
# =========================================================

FROM debian:buster

ENV DEBIAN_FRONTEND=noninteractive
ENV container=docker

# =========================================================
# Fix archived repositories + disable validity check
# =========================================================

RUN echo "deb http://archive.debian.org/debian buster main contrib non-free" > /etc/apt/sources.list && \
    echo "deb http://archive.debian.org/debian-security buster/updates main" >> /etc/apt/sources.list && \
    echo 'Acquire::Check-Valid-Until "0";' > /etc/apt/apt.conf.d/99no-check-valid-until && \
    apt-get update -o Acquire::Retries=5 -o Acquire::http::Timeout=30

# =========================================================
# Install systemd
# =========================================================

RUN apt-get install -y -o Acquire::Retries=5 --no-install-recommends \
    systemd \
    systemd-sysv \
    dbus \
    ca-certificates && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# =========================
# Base system + build deps
# =========================
RUN apt-get update && apt-get install -y -o Acquire::Retries=5 --no-install-recommends \
    git \
    build-essential \
    cmake \
    g++ \
    autoconf \
    automake \
    libtool \
    pkg-config \
    swig \
    python3-dev \
    libxml2 \
    libxml2-dev \
    bison \
    flex \
    libcdk5-dev \
    libusb-1.0-0-dev \
    libaio-dev \
    libfftw3-dev \
    libsctp-dev \
    libboost-program-options-dev \
    libconfig++-dev \
    libboost-all-dev \
    libzmq3-dev \
    libreadline-dev \
    libncurses5-dev \
    libssl-dev \
    wget \
    curl \
    ca-certificates \
    && apt-get clean && \
    rm -rf /var/lib/apt/lists/*

STOPSIGNAL SIGRTMIN+3

VOLUME ["/sys/fs/cgroup"]

# =========================================================
# Base tools
# =========================================================

RUN apt-get update -o Acquire::Retries=5 -o Acquire::http::Timeout=30 && \
    apt-get install -y -o Acquire::Retries=5 --no-install-recommends \
    nano \
    zsh \
    vim \
    wget \
    curl \
    unzip \
    zip \
    telnet \
    fuse \
    gedit \
    mousepad \
    wireshark \
    git \
    git-core \
    inetutils-tools

# =========================================================
# Telecom / SDR build dependencies
# =========================================================

RUN apt-get install -y -o Acquire::Retries=5 --no-install-recommends \
    build-essential \
    gcc \
    g++ \
    make \
    cmake \
    automake \
    autoconf \
    libtool \
    shtool \
    pkg-config \
    ccache \
    doxygen \
    cpufrequtils \
    ethtool \
    gcc-arm-none-eabi \
    dahdi-source

# =========================================================
# Libraries
# =========================================================

RUN apt-get install -y -o Acquire::Retries=5 --no-install-recommends \
    sqlite3 \
    libsqlite3-dev \
    libssl-dev \
    libsctp-dev \
    libfftw3-dev \
    libfftw3-3 \
    libgnutls28-dev \
    libcurl4-gnutls-dev \
    libtalloc-dev \
    libpcsclite-dev \
    libusb-1.0-0 \
    libusb-1.0-0-dev \
    libusb-dev \
    libortp-dev \
    libdbi-dev \
    libdbd-sqlite3 \
    libpcap-dev \
    libc-ares-dev \
    libsofia-sip-ua-glib-dev \
    libgsm1-dev \
    libboost-all-dev \
    libncurses5 \
    libncurses5-dev \
    libmnl-dev \
    libfuse-dev

# =========================================================
# Python environment
# =========================================================

RUN apt-get install -y -o Acquire::Retries=5 --no-install-recommends \
    python3 \
    python3-dev \
    python3-pip \
    python3-mako \
    python3-numpy \
    python3-requests \
    python3-scipy \
    python3-setuptools \
    python3-ruamel.yaml

RUN pip3 install --no-cache-dir smpplib

# =========================================================
# Asterisk
# =========================================================

RUN apt-get install -y -o Acquire::Retries=5 --no-install-recommends \
    asterisk

# =========================================================
# Create source directory
# =========================================================

RUN mkdir -p /opt/src

WORKDIR /opt/src


# =========================================================
# Download USRP Images
# =========================================================
RUN mkdir src && \
cd src && \
git clone --depth 1 -b v3.15.0.0 https://github.com/EttusResearch/uhd && \
cd uhd/host && \
mkdir build && \
cd build && \
cmake .. && \
make -j $(nproc --ignore 1) && \
make install && \
ldconfig && \
uhd_images_downloader


# =========================================================
# libosmocore
# =========================================================

WORKDIR /opt/src

RUN git clone --depth 1 -b 1.7.0 \
    https://gitea.osmocom.org/osmocom/libosmocore

WORKDIR /opt/src/libosmocore

RUN autoreconf -fi && \
    ./configure && \
    make -j$(nproc) && \
    make check || true && \
    make install && \
    ldconfig

# =========================================================
# libosmo-dsp
# =========================================================

WORKDIR /opt/src

RUN git clone https://git.osmocom.org/libosmo-dsp.git

WORKDIR /opt/src/libosmo-dsp

RUN git checkout 551b9752bcd5d3d21bb2df0736b1801bda3d0d10 && \
    autoreconf -i && \
    ./configure && \
    make -j$(nproc) && \
    make install && \
    ldconfig

# =========================================================
# libosmo-abis
# =========================================================

WORKDIR /opt/src

RUN git clone --depth 1 -b 0.6.0 \
    https://gitea.osmocom.org/osmocom/libosmo-abis

WORKDIR /opt/src/libosmo-abis

RUN autoreconf -fi && \
    ./configure && \
    make -j$(nproc) && \
    make check || true && \
    make install && \
    ldconfig

# =========================================================
# libosmo-netif
# =========================================================

WORKDIR /opt/src

RUN git clone --depth 1 -b 0.4.0 \
    https://gitea.osmocom.org/osmocom/libosmo-netif

WORKDIR /opt/src/libosmo-netif

RUN autoreconf -fi && \
    ./configure && \
    make -j$(nproc) && \
    make check || true && \
    make install && \
    ldconfig

# =========================================================
# libosmo-sccp
# =========================================================

WORKDIR /opt/src

RUN git clone --depth 1 -b 1.0.0 \
    https://gitea.osmocom.org/osmocom/libosmo-sccp

WORKDIR /opt/src/libosmo-sccp

RUN autoreconf -fi && \
    ./configure && \
    make -j$(nproc) && \
    make check || true && \
    make install && \
    ldconfig

# =========================================================
# libasn1c
# =========================================================

WORKDIR /opt/src

RUN git clone --depth 1 -b 0.9.31 \
    https://gitea.osmocom.org/cellular-infrastructure/libasn1c

WORKDIR /opt/src/libasn1c

RUN autoreconf -fi && \
    ./configure && \
    make -j$(nproc) && \
    make check || true && \
    make install && \
    ldconfig

# =========================================================
# libsmpp34
# =========================================================

WORKDIR /opt/src

RUN git clone --depth 1 -b 1.13.0 \
    https://gitea.osmocom.org/cellular-infrastructure/libsmpp34

WORKDIR /opt/src/libsmpp34

RUN autoreconf -fi && \
    ./configure && \
    make -j$(nproc) && \
    make check || true && \
    make install && \
    ldconfig

# =========================================================
# osmo-iuh
# =========================================================

WORKDIR /opt/src

RUN git clone --depth 1 -b 0.4.0 \
    https://gitea.osmocom.org/cellular-infrastructure/osmo-iuh

WORKDIR /opt/src/osmo-iuh

RUN autoreconf -fi && \
    ./configure && \
    make -j$(nproc) && \
    make check || true && \
    make install && \
    ldconfig

# =========================================================
# osmo-ggsn
# =========================================================

WORKDIR /opt/src

RUN git clone --depth 1 -b 1.9.0 \
    https://gitea.osmocom.org/cellular-infrastructure/osmo-ggsn

WORKDIR /opt/src/osmo-ggsn

RUN autoreconf -fi && \
    ./configure && \
    make -j$(nproc) && \
    make check || true && \
    make install && \
    ldconfig

# =========================================================
# osmo-sip-connector
# =========================================================

WORKDIR /opt/src

RUN git clone --depth 1 -b 1.3.1 \
    https://gitea.osmocom.org/cellular-infrastructure/osmo-sip-connector

WORKDIR /opt/src/osmo-sip-connector

RUN autoreconf -fi && \
    ./configure && \
    make -j$(nproc) && \
    make check || true && \
    make install && \
    ldconfig

# =========================================================
# osmo-trx
# =========================================================

WORKDIR /opt/src

RUN git clone --depth 1 -b 1.2.0 \
    https://gitea.osmocom.org/cellular-infrastructure/osmo-trx

WORKDIR /opt/src/osmo-trx

RUN autoreconf -fi && \
    ./configure --with-uhd && \
    make -j$(nproc) && \
    make check || true && \
    make install && \
    ldconfig

# =========================================================
# osmo-bts
# =========================================================

WORKDIR /opt/src

RUN git clone --depth 1 -b 1.2.2 \
    https://gitea.osmocom.org/cellular-infrastructure/osmo-bts

WORKDIR /opt/src/osmo-bts

RUN autoreconf -fi && \
    ./configure --enable-trx && \
    make -j$(nproc) && \
    make check || true && \
    make install && \
    ldconfig

# =========================================================
# osmo-pcu
# =========================================================

WORKDIR /opt/src

RUN git clone --depth 1 -b 0.6.0 \
    https://gitea.osmocom.org/cellular-infrastructure/osmo-pcu

WORKDIR /opt/src/osmo-pcu

RUN autoreconf -fi && \
    ./configure && \
    make -j$(nproc) && \
    make check || true && \
    make install && \
    ldconfig

# =========================================================
# openbsc
# =========================================================

WORKDIR /opt/src

RUN git clone --depth 1 -b 1.4.1 \
    https://gitea.osmocom.org/cellular-infrastructure/openbsc

WORKDIR /opt/src/openbsc/openbsc

RUN autoreconf -fi && \
    ./configure \
    --enable-mgcp-transcoding \
    --enable-nat \
    --enable-smpp \
    --enable-osmo-bsc && \
    make -j$(nproc) && \
    make check || true && \
    make install && \
    ldconfig

# =========================================================
# osmo-hlr
# =========================================================

WORKDIR /opt/src

RUN git clone --depth 1 -b 1.0.0 \
    https://gitea.osmocom.org/cellular-infrastructure/osmo-hlr

WORKDIR /opt/src/osmo-hlr

RUN autoreconf -fi && \
    ./configure && \
    make -j$(nproc) && \
    make check || true && \
    make install && \
    ldconfig

# =========================================================
# osmo-sgsn
# =========================================================

WORKDIR /opt/src

RUN git clone --depth 1 -b 1.6.2 \
    https://gitea.osmocom.org/cellular-infrastructure/osmo-sgsn

WORKDIR /opt/src/osmo-sgsn

RUN autoreconf -fi && \
    ./configure && \
    make -j$(nproc) && \
    make check || true && \
    make install && \
    ldconfig

# =========================================================
# Osmocom runtime directories
# =========================================================

RUN mkdir -p /var/lib/osmocom && \
    touch /var/lib/osmocom/hlr.sqlite3

RUN mkdir -p /etc/osmocom

# =========================================================
# Download Osmocom configs
# =========================================================

WORKDIR /etc/osmocom

RUN wget -T 30 --tries=5 \
    https://raw.githubusercontent.com/SitrakaResearchAndPOC/nitb-script-all/main/osmo-trx-uhd.cfg && \
    cat osmo-trx-uhd.cfg

# =========================================================
# Download NITB scripts
# =========================================================


WORKDIR /opt

RUN wget -T 30 --tries=5 \
    https://raw.githubusercontent.com/SitrakaResearchAndPOC/nitb-script-all/main/osmo-nitb-scripts-v3.zip && \
    unzip osmo-nitb-scripts-v3.zip && \
    cd osmo-nitb-scripts && \
    chmod +x install_services.sh && \
    cp config.json /

# =========================================================
# Ports
# =========================================================

EXPOSE 4242/tcp
EXPOSE 4250/tcp
EXPOSE 4251/tcp
EXPOSE 5700/udp
EXPOSE 5701/udp
EXPOSE 5000/udp
EXPOSE 5001/udp

# =========================================================
# Cleanup
# =========================================================

RUN apt-get clean && \
    rm -rf /tmp/* && \
    rm -rf /var/tmp/* && \
    rm -rf /var/lib/apt/lists/*

# =========================================================
# Default workdir
# =========================================================

WORKDIR /opt

# =========================================================
# Start systemd
# =========================================================

CMD ["/sbin/init"]
