# Android Build Environment Setup

## 1. Host Operating System Prerequisites

* **Recommended Distros:** Ubuntu 22.04 LTS / 24.04 LTS, Arch Linux, Fedora 40+.
* **Hardware:** 32 GB RAM minimum, 400 GB NVMe SSD space.

### Required Packages (Debian / Ubuntu)
```bash
sudo apt update && sudo apt install -y \
    git-core gnupg flex bison build-essential zip curl zlib1g-dev \
    libc6-dev-i386 libncurses5 lib32ncurses5-dev x11proto-core-dev \
    libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip \
    fontconfig python3 python-is-python3 libssl-dev bc ccache libarchive-tools lz4
```

---

## 2. Repo Tool Configuration
```bash
mkdir -p ~/.bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/.bin/repo
chmod a+rx ~/.bin/repo
export PATH="${HOME}/.bin:${PATH}"
```
