# WSL

## Building your own WSL 2 kernel with additional drivers

Update WSL:

```pwsh
wsl --update
```

List your distributions.

```pwsh
wsl --list --verbose
```

Verify that your target distribution is version 2;
see [WSL documentation](https://docs.microsoft.com/en-us/windows/wsl/install-win10#set-your-distribution-version-to-wsl-1-or-wsl-2)
for instructions on how to set the WSL version.

Export current distribution to be able to fall back if something goes wrong.

```pwsh
wsl --export <current-distro> <temporary-path>\wsl2-usbip.tar
```

Import new distribution with current distribution as base.

```pwsh
wsl --import wsl2-usbip <install-path> <temporary-path>\wsl2-usbip.tar
```

Run new distribution.

```pwsh
wsl --distribution wsl2-usbip --user <user>
```

Update resources (assuming `apt`, you may need to use `yum` or another package manager).

```bash
sudo apt update
sudo apt upgrade
```

Install prerequisites.

```bash
sudo apt install build-essential flex bison libssl-dev libelf-dev libncurses-dev autoconf libudev-dev libtool bc pahole dwarves
```

> [!NOTE]
> Depending on the drivers you will select below, additional packages may be required.

Clone kernel that matches WSL version. To find the version you can run.

```bash
uname -r
```

The kernel can be found at: <https://github.com/microsoft/WSL2-Linux-Kernel>

Clone the kernel repo, then checkout the branch/tag that matches your kernel version; run `uname -r` to find the kernel version.

```bash
git clone https://github.com/microsoft/WSL2-Linux-Kernel.git
cd WSL2-Linux-Kernel
git checkout linux-msft-wsl-5.10.43.3
gunzip -kc /proc/config.gz |>/dev/null sudo tee .config
sudo make menuconfig
```

These are the necessary additional features in menuconfig.\
Device Drivers -> USB Support\
Device Drivers -> USB Support -> USB announce new devices\
Device Drivers -> USB Support -> USB Modem (CDC ACM) support\
Device Drivers -> USB Support -> USB/IP\
Device Drivers -> USB Support -> USB/IP -> VHCI HCD\
Device Drivers -> USB Support -> USB Mass Storage support

```bash
CORE=$(getconf _NPROCESSORS_ONLN)
sudo make -j $CORE && sudo make modules_install -j $CORE && sudo make install -j $CORE
```

From the root of the repo, copy the image.

```bash
cp arch/x86/boot/bzImage /mnt/c/Users/<user>/usbip-bzImage
```

Create a `.wslconfig` file on `/mnt/c/Users/<user>/` and add a reference to the created image with the following.

```ini
[wsl2]
kernel=c:/users/<user>/usbip-bzImage
```

Your WSL distro is now ready to use!
