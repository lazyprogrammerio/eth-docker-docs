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

### Geth on RISCV64

### Nimbus on RISCV64
