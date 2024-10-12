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

