---
id: OtherArches
title: Other CPU Architectures
sidebar_label: Other CPU Architectures
---

## RISCV64

An Ethereum node can be also run on RISC-V boards, like LicheePI4A.

Supported execution clients:

  * Geth

Supported consensus clients:

  * Nimbus

### Requirements

Currently, this how to has been verifed on Ubuntu 24.04 and Ubuntu 24.10 running on [LicheePI 4A](https://wiki.sipeed.com/hardware/en/lichee/th1520/lpi4a/1_intro.html) and [Sifive](https://www.sifive.com/boards/hifive-unmatched-revb).

Other useful documentation:

  * https://wiki.sipeed.com/hardware/en/lichee/th1520/lpi4a/4_burn_image.html
  * https://www.armbian.com/licheepi-4a
  * https://ubuntu.com/tutorials/how-to-install-ubuntu-on-risc-v-hifive-boards#1-overview
  * https://wiki.ubuntu.com/RISC-V/SiFive%20HiFive%20Unmatched

After installation (see installation docs https://ubuntu.com/download/risc-v), Docker and other packages need to be installed.

```
# install
sudo apt update
sudo apt install --yes docker.io docker-compose-v2 git screen
sudo usermod -aG docker $USER
newgrp docker

# verify installation
docker run hello-world
```

Also, before starting, it is highly recommended to verify the speed of the storage device. The Hifive Unmatched supports an NVME PCIe 3 device and the following IO performance has been seen on a very low tier NVME:

```bash
sudo apt install --yes fio
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 \
   --name=test --filename=test --bs=4k --iodepth=64 \
    --size=1G --readwrite=randrw --rwmixread=75

# read: IOPS=20.5k, BW=80.2MiB/s (84.1MB/s)(768MiB/9568msec)19
# write: IOPS=6860, BW=26.8MiB/s (28.1MB/s)(256MiB/9568msec); 0 zone resets
```

After the initial clone and install while following the docs https://eth-docker.net/Usage/QuickStart#eth-docker-quickstart, we need to define the Docker source values in the .env file before running `ethd config`, for Geth and Nimbus.

### Banana PI

```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 871920D1991BC93C
# add Ubuntu 24.04 RISCV64 ports to be able to install docker-compose-v2

```

### Geth and Nimbus on RISCV64

```
git clone https://github.com/lazyprogrammerio/eth-docker
cd eth-docker
# run ethd install to make sure everything is alright
./ethd install
# run ethd config to create the config .env file
./ethd config
# select holesky

# modify the .env file by adding the following at the end
# https://github.com/lazyprogrammerio/eth-docker/commit/05f34f5b1b712f066bf12fe2aa81bccfc1b7be47
# there is GETH_DOCKER_REPO=lazyprogrammeriow/eth-docker should be GETH_DOCKER_REPO=lazyprogrammerio/eth-docker
# NIM_DOCKERFILE=Dockerfile.source.riscv64.min
# GETH_DOCKER_TAG=geth-alpine
# GETH_DOCKER_REPO=lazyprogrammerio/eth-docker
# GETH_DOCKERFILE=Dockerfile.binary
# run ethd config again to build the images
./ethd config

# start ethd
./ethd start
```

### Lighthouse on RISCV64

```
# install docker.io

docker run -it alpine:edge /bin/sh

# inside the docker container

export OPENSSL_NO_VENDOR=Y
apk update
apk add vim
apk add --no-cache make gcc musl-dev linux-headers git bash git-lfs ca-certificates bash tzdata git curl
apk add rust clang
apk add cargo
apk add openssl
apk add pkgconf openssl-dev
apk add openssl-libs-static
apk add alpine-sdk openssl-dev
apk add cmake

git clone https://github.com/haurog/lighthouse
git clone https://github.com/haurog/rust-libp2p
git clone https://github.com/haurog/utils
cd lighthouse

export RUSTFLAGS="$RUSTFLAGS -L /usr/lib/"
# to continue on SIGSEV errors
# Maybe fixable by:
# https://forum.banana-pi.org/t/banana-pi-f3-with-16-gb-ram-constantly-freezing-solved/18678/30?u=lazyprogrammerio
# Add codegen-units=1 to the Cargo.toml release profile
# [profile.release]
# codegen-units=1
until OPENSSL_LIB_DIR=/usr/lib/ OPENSSL_DIR=/usr/ PROFILE=release make; do sleep 1; done;

# docker run -it lighhouse:binary --version
# Lighthouse v5.2.1-4019e70+
# BLS library: blst
# SHA256 hardware acceleration: false
# Allocator: jemalloc
# Profile: release
# Specs: mainnet (true), minimal (false), gnosis (false)

# Added the lighhouse binary to https://github.com/lazyprogrammerio/eth-docker/releases/tag/v2.11.0.0
# Needs to be copied over to eth-docker/lighthouse directory
# Then do an ./ethd config, choose geth and lighthouse
# Change .env file according to https://github.com/lazyprogrammerio/eth-docker/commit/11be4a6796373951c1c4c9dfaeaff4231c7fc2c1
# Then do another ./ethd config and ./ethd start
```

### For Irradium

See: https://forum.banana-pi.org/t/irradium-based-on-crux-linux-spacemit-banana-pi-bpi-f3-k1-riscv64/17989/14

```
ip route add default via 192.168.1.1 dev eth1
vim /etc/resolv.conf
cd /etc/ports
mv contrib.rsync.inactive contrib.rsync
cd
ports -u
prt-get install wget curl vim
prt-get install screen
# see https://crux.nu/Main/Handbook3-7#ntoc34
# docker
# https://crux.nu/ports/contrib/3.7/docker/Pkgfile

# install go
wget https://go.dev/dl/go1.22.6.linux-riscv64.tar.gz
tar -xzvf go1.22.6.linux-riscv64.tar.gz
mv go /usr/lib/
ln -s /usr/lib/go/bin/go /usr/bin/go
ln -s /usr/lib/go/bin/gofmt /usr/bin/gofmt

cd /usr/ports/contrib/docker
pkgmk -d -i

cd /usr/ports/contrib/docker
pkgmk -d -i

cd /usr/ports/contrib/runc
prt-get install libseccomp
pkgmk -d -i

cd /usr/ports/contrib/containerd
pkgmk -d -i

cd /usr/ports/contrib/cgroupfs-mount
pkgmk -d -i
cgroupfs-mount

# start dockerd
dockerd
```

### BFi3 overclocking to 1.8 Ghz - DO IT AT YOUR OWN RISK

```
# On An Ubuntu 24.04 AMD64
sudo apt update
sudo apt-get install gcc-riscv64-linux-gnu

git clone https://github.com/BPI-SINOVOIP/pi-linux --single-branch -b linux-6.1.15-k1-dev --depth 1
cd pi-linux
# apply patch
alias rvmake='make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- -j60 '
rvmake k1_defconfig
rvmake dtbs
# mount the /boot partition from Bianbu, usually sda5 to /tmp/bianbu
# backup the old dtb
mkdir /tmp/bianbu/spacemit/6.1.15/old-dtb
mv /tmp/bianbu/spacemit/6.1.15/*.dtb /tmp/bianbu/spacemit/6.1.15/old-dtb/
# copy the new dtb
sudo cp arch/riscv/boot/dts/spacemit/*.dtb /tmp/bianbu/spacemit/6.1.15/
```

```patch
diff --git a/arch/riscv/boot/dts/spacemit/k1-x_opp_table.dtsi b/arch/riscv/boot/dts/spacemit/k1-x_opp_table.dtsi
index 6672b4816..22d9e3cd9 100644
--- a/arch/riscv/boot/dts/spacemit/k1-x_opp_table.dtsi
+++ b/arch/riscv/boot/dts/spacemit/k1-x_opp_table.dtsi
@@ -11,6 +11,14 @@ clst_core_opp_table0: opp_table0 {
                clock-names = "ace0","ace1","tcm","cci","pll3";
                cci-hz = /bits/ 64 <614000000>;

+               opp1800000000 {
+                       opp-hz = /bits/ 64 <1800000000>, /bits/ 64 <1800000000>;
+                       tcm-hz = /bits/ 64 <900000000>;
+                       ace-hz = /bits/ 64 <900000000>;
+                       opp-microvolt = <1160000>;
+                       clock-latency-ns = <200000>;
+               };
+
                opp1600000000 {
                        opp-hz = /bits/ 64 <1600000000>, /bits/ 64 <1600000000>;
                        tcm-hz = /bits/ 64 <800000000>;
```


