# Setting up Looking Glass
> Homepage: https://looking-glass.io/

Looking Glass allows you to connect to your VM's display with minimal latency and lag. It works by creating a shared memory space between the guest (VM) and host. The video stream does not go trough a network socket (like with RDP/VNC/etc.), but the client can just simply read this shared memory, allowing almost instant access to the video stream without any compression or image processing.

LG technically requires an HDMI dummy plug, but in this guide I'll assume you don't have one. **You don't need to buy anything. There is a workaround!**

... TODO ...