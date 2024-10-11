# Installing QEMU and Virtual Machine Manager

> This guide is based on the [Fedora Virtualization - Getting started guide](https://docs.fedoraproject.org/en-US/quick-docs/virtualization-getting-started/).

Run the following command to *install the needed virtualization tools*:
```sh
sudo dnf group install --with-optional virtualization
```

After the installation is done, run the following command to *enable the virtualization service to run at boot* and *start it up immediately*:
```sh
sudo systemctl enable --now libvirtd
```

You should restart your computer after doing this.

To verify that virtualization modules are loaded, run the following command:
```sh
lsmod | grep kvm
```

In my case, I get the following:
```
kvm_amd               221184  0  
kvm                  1445888  1 kvm_amd  
ccp                   180224  1 kvm_amd
```
> You should have something similar.

#### Done, continue with the [next part](WINDOWS_SETUP.md)!