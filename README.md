# wsl-vhd

Automagically create/format/mount VHD disk images in WSL on boot.

## Why?

To date, this is the best method i've found of decoupling user data from WSL distributions.
This is somewhat comparable to creating `/home` as a separate partition on a native Linux box.
Personally, I store all projects and code in a VHD, which allows me to:

1) Remove/reinstall a distribution without a tedious backup process
2) Share the same files across all running WSL distributions
3) Play with btrfs :)

## Windows Requirements

 - Windows 11 21H2 or higher.
 - WSL installed from the Microsoft Store: <https://devblogs.microsoft.com/commandline/a-preview-of-wsl-in-the-microsoft-store-is-now-available/>

## Install

Debian/Ubuntu: `sudo apt install qemu-utils`

Redhat/Fedora: `sudo dnf install qemu-img`

```bash
cd ~
git clone "https://github.com/kmmiles/wsl-vhd.git"
cd wsl-vhd
sudo ln -sf "$(pwd)/wsl-vhd" /usr/local/bin/wsl-vhd
```

## Run on boot

Add a line like this to `/etc/wsl.conf`:

```
[boot]
command = wsl-vhd -v use -u myuser /mnt/c/wsl-vhd/code.vhdx ext4 10000 /tmp/wsl-vhd.log 2>&1
```
Replacing `myuser` with your username and any other adjustments. Then shutdown WSL with `wsl --shutdown` and restart.
This example will create a 10GB VHD at `C:\wsl-vhd\code.vhdx`, which will be accessible in WSL at `/mnt/wsl/code`.
On first boot, it will automatically be created and formatted for you. Multiple images can be handled by tacking on more
arguments e.g.

```
[boot]
command = wsl-vhd -v use -u myuser \
 /mnt/c/wsl-vhd/code.vhdx ext4 10000 \
 /mnt/c/wsl-vhd/code2.vhdx btrfs 20000 \
 /mnt/c/wsl-vhd/code3.vhdx ntfs 30000 > \
 /tmp/wsl-vhd.log 2>&1
```

Also note that the sizes and filesystem types can be adjusted per image. If something goes wrong, a log will stored at `/tmp/wsl-vhd.log`. If you need more info, replace `-v` with `-vv` (or `-vvv` for the verbosiest among us)

## Manually invoking

The script includes a CLI for creating, mounting, unmounting, etc. Check `wsl-vhd -h`.
