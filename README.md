# Fedora QEMU GPU passthrough Guide
This guide will help you set up GPU passthrough for a QEMU/KVM virtual machine on Fedora.

This guide has been tested with a **Lenovo Legion Pro 7 Gen 8** (*16ARX8H*) laptop running Fedora 40 w/ KDE.

🖥️ Device specs:
- AMD Ryzen 9 7945HX
- NVIDIA GeForce RTX4090 Mobile

This guide is broken up into 3 parts:
- Installing QEMU and Virtual Machine Manager
- Creating a Windows 10 VM
- Setting up your kernel for GPU passthrough

### Getting started
In order to make this work, there are a few requirements:
1. Your PC/laptop must have **at least 2 GPUs**.
2. Your hardware configuration must be such that your dGPU is in it's own IOMMU group.

### Dealing with IOMMU groups
You can read about IOMMU groups [here](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-iommu-deep-dive). To check your computer's IOMMU groups, use [this](https://gist.github.com/flungo/428c374c040de1d0a30fd4a593d39040) script, or [this](https://gist.github.com/r15ch13/ba2d738985fce8990a4e9f32d07c6ada) one which has a prettier output.

In my case:
```sh
$ ./iommu.sh
...
Group 14:       [10de:2757] [R] 01:00.0  VGA compatible controller                GN21-X11 [GeForce RTX 4090 Laptop GPU]  
                [10de:22bb]     01:00.1  Audio device                             Device 22bb
...
```

The first device (`10de:2757`) is the actual GPU, while the second one (`10de:22bb`) is it's sound card. The hexadecimal numbers inside the square brackets (`[`/`]`) indicate the PCIe ID of the device. You'll need these for later.

#### Problems
Your hardware may be designed in a way that the IOMMU group that contains your dGPU may also contain other unrelated devices. This can make passthrough more difficult, **but not impossible**.

Fortunately, the PCI Express specification includes support for Access Control Services (*ACS*). ACS allows you to isolate your PCIe devices, breaking them up into multiple IOMMU groups.

To use ACS, you must use a special Linux kernel that has it enabled. The simplest solution is to just use a distro that comes with an ACS kernel by default - [Nobara Linux](https://nobaraproject.org/). Nobara is a modified Fedora disto, that comes with some improvements, mainly for gaming, but regardless, it's just as usable as regular Fedora.

##### ⚠️ {WARNING} Installing Nobara alongside another distro
Nobara will likely overwrite your EFI partition, making it impossible to boot into another Linux install. You should install it onto a different SSD, or at least back up your current one with something like [Clonezilla](https://clonezilla.org/).

#### Done, continue with the [next part](VIRTUALIZATION_SETUP.md)!