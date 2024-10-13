# Fedora QEMU GPU passthrough Guide
This guide will help you set up GPU passthrough for a QEMU/KVM virtual machine on Fedora.

This guide has been tested with a [**Lenovo Legion Pro 7 Gen 8** (*16ARX8H*)](https://psref.lenovo.com/Product/Legion/Legion_Pro_7_16ARX8H) laptop running Fedora 40 w/ KDE.

🖥️ Device specs:
- AMD Ryzen 9 7945HX
- NVIDIA GeForce RTX4090 Mobile

This guide is broken up into 4 parts:
- [Installing QEMU and Virtual Machine Manager](VIRTUALIZATION_SETUP.md)
- [Creating a Windows 10 VM](WINDOWS_SETUP.md)
- [Setting up your kernel for GPU passthrough](KERNEL_SETUP.md)
- *(Optional)* [Setting up Looking Glass](LOOKING_GLASS_SETUP.md)

### Requirements
In order to make this work, there are a few requirements:
1. You must have **at least 2 GPUs**.
2. Your CPU should have **at least 4 cores**.

### Before starting
Read these two pages:
1. [IOMMU Groups](IOMMU_GROUPS.md)
2. [Installing proprietary NVIDIA Drivers](NVIDIA_DRIVERS.md)
	1. *Optional!*
	2. *Only for new Fedora users, who want to use their dGPU "normally" - not just for VMs.*

#### Done, continue with the [next part](VIRTUALIZATION_SETUP.md)!