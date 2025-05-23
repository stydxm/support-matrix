---
sys: archlinux
sys_ver: null
sys_var: null

status: basic
last_update: 2024-12-04
---

# Archlinux AWOL Nezha D1 Test Report

## Test Environment

### Operating System Information

- Base Image: Ubuntu 24.10: [ubuntu-24.10](https://ubuntu.com/download/risc-v)
  - Or any arbitrary image for D1
- Rootfs：[archriscv-2024-09-22.tar.zst](https://archriscv.felixc.at/images/archriscv-2024-09-22.tar.zst)
- Reference Installation Document: https://github.com/felixonmars/archriscv-packages/wiki/RV64-%E6%9D%BF%E5%AD%90%E6%9B%B4%E6%8D%A2-rootfs-%E6%8C%87%E5%8D%97

### Hardware Information

- Nezha D1
- A Type-C Power Cable
- A UART to USB Debugger
- SD Card

## Installation Steps

### Obtain images and rootfs

Get the base images and rootfs. You can use arbitrary any images for D1 as the base images.

We have used Debian images and Ubuntu images and know that they work well. Below we use the Ubuntu images as an example.

```bash
wget https://archriscv.felixc.at/images/archriscv-2024-09-22.tar.zst
xz -kd ubuntu-24.10-preinstalled-server-riscv64+nezha.img.xz
mkdir rfs
tar -xf archriscv-2024-09-22.tar.zst -C rfs
mkdir mnt
```

### Replace the rootfs

Replace the rootfs with the one from the Ubuntu image. Change the following mount points to the correct rootfs for your image.

```bash
sudo losetup -f
sudo losetup -P /dev/loopX ubuntu-24.10-preinstalled-server-riscv64+nezha.img.xz
sudo mount /dev/loopXp1 mnt
cd mnt
sudo mkdir old
sudo mv etc home media mnt opt root srv var usr old/
sudo cp -r ../rfs/{etc,home,mnt,opt,root,srv,var,usr} .
sudo cp -r old/usr/lib/firmware usr/lib/
sudo cp -r old/usr/lib/modules/ usr/lib/
rm etc/fstab
sudo rm etc/fstab
sudo cp -r old/etc/fstab etc/fstab
```

Don't forget to clean up the files.

```bash
cd ..
sudo umount mnt
sudo losetup -d /dev/loopX
```

### Flash the image

Flash the image to the SD card.

```bash
sudo wipefs -a /dev/sdX
sudo dd if=ubuntu-24.10-preinstalled-server-riscv64+nezha.img.xz of=/dev/sdX bs=4M status=progress
```

### Logging into the System

Logging into the system via the serial port.

Default Username: `root`
Default Password: `archriscv`

## Expected Results

The system should boot normally and allow login via the onboard serial port.

## Actual Results

The system booted successfully and login via the onboard serial port was also successful.

### Boot Log

Screen recording (from flashing the image to logging into the system):
[![asciicast](https://asciinema.org/a/G3j3MjoOZ8rcTD28kfMLDao6a.svg)](https://asciinema.org/a/G3j3MjoOZ8rcTD28kfMLDao6a)

```log
archlinux login: root
Password: 
[root@archlinux ~]# cat /etc/os-release 
NAME="Arch Linux"
PRETTY_NAME="Arch Linux"
ID=arch
BUILD_ID=rolling
ANSI_COLOR="38;2;23;147;209"
HOME_URL="https://archlinux.org/"
DOCUMENTATION_URL="https://wiki.archlinux.org/"
SUPPORT_URL="https://bbs.archlinux.org/"
BUG_REPORT_URL="https://gitlab.archlinux.org/groups/archlinux/-/issues"
PRIVACY_POLICY_URL="https://terms.archlinux.org/docs/privacy-policy/"
LOGO=archlinux-logo
[root@archlinux ~]# uname -a
Linux archlinux 6.11.0-8-generic #8.1-Ubuntu SMP PREEMPT_DYNAMIC Tue Oct  1 11:40:56 UTC 2024 riscv64 GNU/Linux
[root@archlinux ~]# cat /proc/cpuinfo 
processor       : 0
hart            : 0
isa             : rv64imafdc_zicntr_zicsr_zifencei_zihpm_zca_zcd
mmu             : sv39
uarch           : thead,c906
mvendorid       : 0x5b7
marchid         : 0x0
mimpid          : 0x0
hart isa        : rv64imafdc_zicntr_zicsr_zifencei_zihpm_zca_zcd
```

## Test Criteria

Successful: The actual result matches the expected result.

Failed: The actual result does not match the expected result.

## Test Conclusion

Test successful
