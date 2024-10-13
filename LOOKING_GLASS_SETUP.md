# Setting up Looking Glass
> Homepage: https://looking-glass.io/

Looking Glass allows you to connect to your VM's display with minimal latency and lag. It works by creating a shared memory space between the guest (VM) and host. The video stream does not go trough a network socket (like with RDP/VNC/etc.), but the client can just simply read this shared memory, allowing almost instant access to the video stream without any compression or image processing.

LG technically requires an HDMI dummy plug, but in this guide I'll assume you don't have one. **You don't need to buy anything. There is a workaround!**

### Compiling LG
Install the dependencies for Looking Glass:
```sh
sudo dnf install cmake gcc gcc-c++ libglvnd-devel fontconfig-devel spice-protocol make nettle-devel \
            pkgconf-pkg-config binutils-devel libXi-devel libXinerama-devel libXcursor-devel \
            libXpresent-devel libxkbcommon-x11-devel wayland-devel wayland-protocols-devel \
            libXScrnSaver-devel libXrandr-devel dejavu-sans-mono-fonts dkms kernel-devel kernel-headers
```

Install dependencies for audio support:
```sh
sudo dnf install pipewire-devel pulseaudio-libs-devel libsamplerate-devel
```

Download the LG source code from [here](https://looking-glass.io/artifact/stable/source). Download and extract this archive to your *Downloads* folder.
```sh
cd ~/Downloads
wget https://looking-glass.io/artifact/stable/source -O looking_glass_src.tar.gz
mkdir looking_glass_src
tar -xvf ~/Downloads/looking_glass_src.tar.gz -C looking_glass_src
```

Go to the LG client source directory and create a build folder in it.
```sh
cd looking_glass_src/looking-glass-B6/client
mkdir build
cd build
```

Generate the build files:
```sh
cmake ../
```

Compile the client:
```sh
make -j
```
> `-j` will make use of all CPU threads. You can limit this by specifying a number after it. For example, if you want 8 threads, use `-j8`.

The compilation shouldn't take longer than 20-30 seconds. After it is compiled, copy the binary to `/usr/local/bin`:
```sh
sudo cp looking-glass-client /usr/local/bin
```

Test if it works:
```sh
looking-glass-client --help
```

### Setting up VM
In Virtmanager, open your VM, go the the `Memory` tab and check `Enabled shared memory`.

... TODO ...