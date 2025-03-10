# DirectX on Linux: DXVK

**THIS GUIDE WAS WRITTEN WHEN DXVK FIRST CAME OUT AND IS NOW OBSOLETE**

A HOWTO for DXVK on Ubuntu with Lutris

There have been several recent advances that make DirectX on Linux playable with good performance.  DirectX previously ran on Wine DirectX 9 with poor performance, frequent crashes and severe stuttering while loading shaders.  doitsujin's DXVK enabled DX11 support which dramatically improved performance and eliminated crashes.  Wine ESYNC further improved performance by reducing CPU overhead.  All of these pieces together (thanks to Lutris) enable a playable DirectX!  A Vulkan-capable DX11 NVIDIA GPU is required as DXVK currently doesn't support AMD (though it might work).

This is written for DXVK version .65 on Ubuntu 18.04 but DXVK is new and evolving rapidly so it will likely be obsolete by the time i click commit.

## Requirements:

* [Vulkan-capable DirectX11 NVIDIA GPU](https://en.wikipedia.org/wiki/Vulkan_(API)#Compatibility)
* NVIDIA drivers version 396.24+
* Wine >= 3.5 (You can install via this HOWTO)
* [Wine dependencies](https://github.com/lutris/lutris/wiki/Wine) - Installing latest Wine-Staging into your system to pull in dependencies is recommended.
* Mesa 18.1.2
* root access and comfort using PPAs
* 64bit Ubuntu 18.04
* A standalone-enabled PoE account (you may need to PM support via the forums to get a standalone password)


# Basic HOWTO steps
0.  Install or update your NVIDIA drivers to version 396.24 or newer
1.  Install Vulkan 
```
sudo apt install libvulkan1 libvulkan1:i386 
```
2.  Install Wine Staging via PPA. Lutris doesn't install it as a dependency and doesn't document that you need Wine as a separate installation (https://lutris.net/downloads/). Don't just 'apt install' it! Our bleeding-edge setup needs the 'staging' version from the offical Wine PPA: https://wiki.winehq.org/Ubuntu
3.  Install Lutris from their PPA https://lutris.net/downloads/
4.  Install the Lutris runners.  The offical instructions are here (https://github.com/lutris/lutris/wiki/How-to:-Esync) but I'll add some comments.  The 'versions' image is rather cryptic. Access that dialog by launching Lutris and in the GUI click Lutris then 'manage runners' and scroll down to Wine and click 'manage versions'. You can install the different versions by single-clicking on the box and waiting for the download to start (the box doesn't get checked...just wait). This takes a while to download and you can download more than one at a time.
5.  Install your DirectX applications via Lutris
6.  Install the patched DXVK.  
Download the release binaries from BlazeKI: https://github.com/BlazeKl/dxvk-async-hack/releases and then unpack them to a custom location that Lutris is able to find.  
7.  Configure the Lutris runner 
8.  Set cpu to performance (per DXVK's documentation):
```
sudo apt-get install cpufrequtils
echo 'GOVERNOR="performance"' | sudo tee /etc/default/cpufrequtils
sudo systemctl disable ondemand
```
9.  Set env vars in the Lutris interface for your application's configuration (Lutris->Right-Click PoE->Configure->System Options->Show Advanced Options->Environment variables):
```
DXVK_USE_PLACEHOLDER_SHADERS=1
DXVK_USE_PIPECOMPILER=1
WINEESYNC=1
```
10.  Done! You can adjust in-app options for your own performance preference but some people report that doing so can resurrect the shader compilation problems.




# Tips for compiling your own DXVK+patch
This section is a combination of the documentation from DXVK and associated patches.

## Requirements:
- [wine 3.50](https://www.winehq.org/) or newer
- [Meson](http://mesonbuild.com/) build system (at least version 0.43)
- [MinGW64](http://mingw-w64.org/) compiler and headers (requires threading support)
- [glslang](https://github.com/KhronosGroup/glslang) compile

1.  Setup your build environment
```
sudo update-alternatives --config x86_64-w64-mingw32-gcc
sudo update-alternatives --config x86_64-w64-mingw32-g++
sudo update-alternatives --config i686-w64-mingw32-gcc
sudo update-alternatives --config i686-w64-mingw32-g++
```
```
sudo apt install mingw-w64 mingw-w64-common mingw-w64-i686-dev mingw-w64-tools mingw-w64-x86-64-dev ninja-build 
meson 
```
2. Download DXVK src and the PoE patch, then apply the patch
```
git clone https://github.com/doitsujin/dxvk.git
git clone https://github.com/BlazeKl/dxvk-async-hack/blob/master/pipeline.patch
cd dxvk
patch -Np1 -i ../pipeline.patch
```
3.  Compile DXVK.  Two options, each with varying build success stories.

3.1
```
./package-release.sh master /your/target/directory --no-package
```
3.2
```
meson --cross-file build-win64.txt --prefix /your/dxvk/directory build.w64
cd build.w64
meson configure 
ninja
ninja install
```

4.  Install your compiled DLLs where Lutris can find them.  Example: ~/.local/share/lutris/runtime/dxvk/<here>
