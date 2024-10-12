# Kernel setup
> This guide is based on [@firelightning13](https://gist.github.com/firelightning13)'s [guide](https://gist.github.com/firelightning13/e530aec3e3a4e15885a10f6c4b7ae021) as well as [@paul-vd](https://gist.github.com/paul-vd)'s [guide](https://gist.github.com/paul-vd/5328d8eb2c626dff36ee143da2e85179).

This part involves adding the necessary kernel parameters for enabling KVM and the [`vfio-pci`](https://www.kernel.org/doc/html/v5.6/driver-api/vfio.html) driver.

### Ways to add kernel parameters
There are several way's to do this. In most tutorials you'll see people suggesting to edit some kind of a global GRUB configuration. **If you're on Fedora, I highly recommend against doing this.** Fedora has a nice CLI tool ([`grubby`](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/managing_monitoring_and_updating_the_kernel/assembly_making-persistent-changes-to-the-grub-boot-loader_managing-monitoring-and-updating-the-kernel)) for safely managing GRUB entries.

If you have already done a kernel update before, you might already have at least 2 kernels on your Fedora install. You should see these in the GRUB bootloader when booting. You can apply the needed kernel argument for only of them, and if something goes wrong, you can just select another one, and fix your mistakes.

I'll show you how to duplicate the latest kernel option in GRUB, and give it the needed kernel arguments. This way, you can choose between a normal boot (with a functioning dGPU) and another one with dGPU passthrough configured (so the dGPU will be unusable for anything other that VMs).

### Creating a duplicate boot option
First off, you need to know what boot options your have first. You can do that by running this command:
```sh
sudo grubby --info=ALL
```
> **Note:** `ALL` *must be uppercase*! This is case-sensitive.

Example output:
```
index=0
kernel="/boot/vmlinuz-6.10.12-200.fc40.x86_64"
args="ro rootflags=subvol=root00 rhgb quiet rd.driver.blacklist=nouveau modprobe.blacklist=nouveau rd.driver.blacklist=nouveau modprobe.blacklist=nouveau"
root="UUID=f2d945db-0ed1-46d6-84d7-8e8247be66ab"
initrd="/boot/initramfs-6.10.12-200.fc40.x86_64.img"
title="Fedora Linux (6.10.12-200.fc40.x86_64) 40 (KDE Plasma)"
id="3fed63c9fc1d4662aeeb2944e8df3554-6.10.12-200.fc40.x86_64"
index=1
kernel="/boot/vmlinuz-6.10.11-200.fc40.x86_64"
args="ro rootflags=subvol=root00 rd.driver.blacklist=nouveau modprobe.blacklist=nouveau rhgb quiet rd.driver.blacklist=nouveau modprobe.blacklist=nouveau"
root="UUID=f2d945db-0ed1-46d6-84d7-8e8247be66ab"
initrd="/boot/initramfs-6.10.11-200.fc40.x86_64.img"
title="Fedora Linux (6.10.11-200.fc40.x86_64) 40 (KDE Plasma)"
id="3fed63c9fc1d4662aeeb2944e8df3554-6.10.11-200.fc40.x86_64"
index=2
kernel="/boot/vmlinuz-6.9.12-200.fc40.x86_64"
args="ro rootflags=subvol=root00 rd.driver.blacklist=nouveau modprobe.blacklist=nouveau rhgb quiet rd.driver.blacklist=nouveau modprobe.blacklist=nouveau"
root="UUID=f2d945db-0ed1-46d6-84d7-8e8247be66ab"
initrd="/boot/initramfs-6.9.12-200.fc40.x86_64.img"
title="Fedora Linux (6.9.12-200.fc40.x86_64) 40 (KDE Plasma)"
id="3fed63c9fc1d4662aeeb2944e8df3554-6.9.12-200.fc40.x86_64"
index=3
kernel="/boot/vmlinuz-0-rescue-3fed63c9fc1d4662aeeb2944e8df3554"
args="ro rootflags=subvol=root00 rd.driver.blacklist=nouveau modprobe.blacklist=nouveau rhgb quiet rd.driver.blacklist=nouveau modprobe.blacklist=nouveau"
root="UUID=f2d945db-0ed1-46d6-84d7-8e8247be66ab"
initrd="/boot/initramfs-0-rescue-3fed63c9fc1d4662aeeb2944e8df3554.img"
title="Fedora Linux (0-rescue-3fed63c9fc1d4662aeeb2944e8df3554) 40 (KDE Plasma)"
id="3fed63c9fc1d4662aeeb2944e8df3554-0-rescue"
```

You can see that I already have 4 boot options. 3 different kernels and a rescue option. The latest kernel version is `6.10.12`, so I'll work with that.

To duplicate the latest kernel option, run the following command. You can name the option whatever you want, I'll just add a `[KVM GPU Passthrough]` suffix at the end
```sh
sudo grubby \
	--grub2 \
	--add-kernel=/boot/vmlinuz-6.10.12-200.fc40.x86_64 \
	--title="Fedora Linux (6.10.12-200.fc40.x86_64) 40 (KDE Plasma) [KVM GPU Passthrough]" \
	--initrd=/boot/initramfs-6.10.12-200.fc40.x86_64.img \
	--copy-default
```

Once this is done, this new option should have index 0 and will be set as the default boot option. Now we'll add the necessary kernel arguments.

```sh
sudo grubby \
	--update-kernel=0 \
	--args="rd.driver.blacklist=nouveau,nvidia,nvidiafb,nvidia-gpu modprobe.blacklist=nouveau,nvidia,nvidiafb,nvidia-gpu"
```

This will add the following 2 kernel arguments:
- `rd.driver.blacklist=nouveau,nvidia,nvidiafb,nvidia-gpu`
	- Blacklists (blocks) the open-source NVIDIA driver `nouveau` as well as the proprietary ones when the kernel is loaded.
- `modprobe.blacklist=nouveau,nvidia,nvidiafb,nvidia-gpu`
	- Prevents the kernel from loading the open-source NVIDIA driver `nouveau` as well as the proprietary ones when the kernel is booted and starts looking for loadable modules.
For more information, click [here](https://stackoverflow.com/a/66111941).

Next, you need to figure out your GPU's PCIe ID. Please refer to the *Dealing with IOMMU groups* section in the [README](README.md).

We'll add the following arguments:
- `video=efifb:off`
	- Disables the `efifb` (EFI Framebuffer) display driver. This prevents it from "stealing" the dGPU.
- `amd_iommu=on` / `intel_iommu=on`
	- Enables the AMD/Intel IOMMU driver.
	- **Use the right `intel`/`amd` parameter according to your CPU.**
	- *On AMD systems, this should be on by default, but you should still have it.*
- `rd.driver.pre=vfio-pci`
	- Forcibly load the `vfio-pci` driver.
- `kvm.ignore_msrs=1`
	- Makes VMs ignore errors when they try to access hardware registers that aren’t actually defined for that processor.
- `vfio-pci.ids=XXXX:XXXX,YYYY:YYYY`
	- Tells the `vfio-pci` driver to use the specified PCIe devices.
	- **Make sure to replace the IDs with your GPU's ID. One ID should be your dGPU's ID and the other one should be it's audio controller.**
	- **The IDs must be comma (`,`) separated, without spaces in between.**

Let's add them:
```sh
sudo grubby \
	--update-kernel=0 \
	--args="video=efifb:off amd_iommu=on rd.driver.pre=vfio-pci kvm.ignore_msrs=1 vfio-pci.ids=10de:2757,10de:22bb"
```

**That's all!** However, I don't want this new boot option to be the default, so I'll set index 1 to be the default. This option had index 0 before.

```sh
sudo grubby --set-default-index=1
```

Now, when your reboot, you should see the new option in the GRUB bootloader. **Please boot using this option before continuing!**

### Verifying the `vfio-pci` driver
Run the following command:
```sh
lspci -vvv -s 01:00.0
```

You should have an output like this:
```
01:00.0 VGA compatible controller: NVIDIA Corporation GN21-X11 [GeForce RTX 4090 Laptop GPU] (rev a1) (prog-if  
00 [VGA controller])  
       Subsystem: Lenovo Device 3c74  
       Physical Slot: 0  
       Control: I/O+ Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx+  
       Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-  
       Latency: 0  
       Interrupt: pin A routed to IRQ 126  
       IOMMU group: 14  
       Region 0: Memory at d0000000 (32-bit, non-prefetchable) [size=16M]  
       Region 1: Memory at 3ff800000000 (64-bit, prefetchable) [size=16G]  
       Region 3: Memory at 3ffc00000000 (64-bit, prefetchable) [size=32M]  
       Region 5: I/O ports at 4000 [size=128]  
       Expansion ROM at d1080000 [virtual] [disabled] [size=512K]  
       Capabilities: <access denied>  
       Kernel driver in use: vfio-pci  
       Kernel modules: nouveau, nvidia_drm, nvidia
```

What you need to look for, is the `Kernel driver in use`. This should be `vfio-pci`. If it's not, this could potentially cause issues later.

### Setting up your VM
Firstly, open *Virtual Machine Manager*, and select your desired virtual machine. Go to the hardware options tab.

1. Under `Display Splice` make sure you have the following options set:
	1. **Type:** `Splice Server`
	2. **Listen type:** `Address`
	3. **Address:** `Hypervisor default`
	4. **Port:** `Auto`
	5. **Password:** *Not set*
	6. **OpenGL:** `Disabled`
2. Under `Video ...` make sure you have the following options set:
	1. **Model:** `VGA`
3. Click `Add Hardware`, select `Input` and set type to `USB Mouse`.
4. Repeat the last step, but choose `USB Keyboard` instead.
5. *If you have a virtual sound card, you can optionally remove it.*
6. Click `Add Hardware`, select `PCI Host Device` and select your dGPU.
7. Repeat the same step for your dGPU's sound card.

![[selecting_dgpu.png]]

After this, you should have both devices attached to your VM.
![[devices_preview.png]]

You can now start the virtual machine.

**Note that you will likely not see your GPU in Device Manager on Windows.** This is normal. Just install your NVIDIA drivers like you normally would, and it will appear normally.