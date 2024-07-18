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

After installation (see installation docs https://ubuntu.com/download/risc-v), Docker and other packages need to be installed.

```
# install
sudo apt update
sudo apt install --yes docker.io docker-compose-v2
sudo usermod -aG docker $USER
newgrp docker

# verify installation
docker run hello-world
```

### Geth on RISCV64

### Nimbus on RISCV64
