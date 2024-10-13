# Installing NVIDIA Drivers
> This guide is based on [How to Set Nvidia as Primary GPU on Optimus-based Laptops](https://docs.fedoraproject.org/en-US/quick-docs/set-nvidia-as-primary-gpu-on-optimus-based-laptops/) by Fedora Project.

> ℹ️ If you're using Nobara, please use the built-in GUI installer.

Fedora currently has a new and old guide to do this. However I find the older one better, so we'll follow that. Also note that this guide is for KDE Plasma, but on GNOME, the process is pretty much the same. **This guide does not involve setting the dGPU as primary GPU, as mentioned in the guide linked above!** We just want to install a driver, nothing else.
#### Enable the NVIDIA driver repo
1. Open Konsole/Terminal.
2. Run `sudo dnf upgrade`. Make sure to update everything before continuing.
3. Open Discover, and go to Settings.
4. Under `Fedora Linux 40 (KDE Plasma)` enable the `RPM Fusion for Fedora 40 - Nonfree - NVIDIA Driver` repository.

#### Refresh repositories
Run the following command to refresh available packages:
```sh
sudo dnf upgrade --refresh
```

#### Installing the driver
Run the following command:
```sh
sudo dnf install gcc kernel-headers kernel-devel akmod-nvidia xorg-x11-drv-nvidia xorg-x11-drv-nvidia-libs xorg-x11-drv-nvidia-libs.i686
```

**Now wait at least 10 minutes until the modules load up! Do not continue without waiting!**

#### Compile the driver
Run the following two commands to *rebuild the kernel module (driver) for all installed kernels* and *regenerate initramfs images*.
```sh
sudo akmods --force
sudo dracut --force
```

#### Note
If you followed the guide linked above, **DO NOT perform Step #8 and later!** That guide is for X11, not Wayland. These steps are unnecessary anyway.

#### Done, go back to the [README](README.md)!