# wsl-vhd-bash

Store stuff (like your code) in a virtual disk, with the filesystem of your choice (ext4, btrfs, ntfs, exfat, etc).
This virtual disk can be automatically mounted and shared between all WSL2 distributions.

*original blogpost: <https://kmmiles.github.io/wsl/vhd/linux/2022/12/20/wsl-vhd.html>*

## Requirements

 - Windows 11 21H2 or higher.
 - WSL installed from the Microsoft Store: <https://devblogs.microsoft.com/commandline/a-preview-of-wsl-in-the-microsoft-store-is-now-available/>

## Install dependencies

### Minimum

Debian/Ubuntu: `sudo apt install util-linux qemu-utils`

Redhat/Fedora: `sudo dnf install util-linux qemu-img`

### With additional filesystems

Adds support for `btrfs`, `ntfs`, `exfat`, `vfat`, `fat`, `msdos`: 

Debian/Ubuntu: `sudo apt install util-linux qemu-utils btrfs-progs ntfs-3g exfat-utils exfat-fuse dosfstools`

Fedora: 
```bash
dnf install -y \
  https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
sudo dnf install util-linux qemu-img btrfs-progs ntfsprogs exfatprogs fuse-exfat dosfstools
```

> Use `wsl-vhd filesystems` to display all filesystems your machine supports.

## Install

Either download the script directly to `/usr/local/bin` and give it execute perms:

```bash
curl -Ls 'https://raw.githubusercontent.com/kmmiles/wsl-vhd-bash/main/wsl-vhd' | sudo tee /usr/local/bin/wsl-vhd > /dev/null && sudo chmod +x /usr/local/bin/wsl-vhd
```

Or check out the project and symlink it:

```bash
cd ~ && git clone "https://github.com/kmmiles/wsl-vhd-bash.git" && sudo ln -sf "$(pwd)/wsl-vhd-bash/wsl-vhd /usr/local/bin/wsl-vhd
```

> `wsl-vhd` just needs to exist somewhere that's included in both your and the `root` users `$PATH`.

## Configuration

Create and add the following to `/etc/wsl-vhd.conf`:

```
[C:\wsl-vhd\test-ext4.vhdx]
fstype=ext4
size_in_mb=100
```

Now run `wsl-vhd up`.

- `C:\wsl-vhd\test-ext4.vhdx` will be created, formatted and mounted at `/mnt/wsl/test-ext4`.
- The mountpoint will be given read/write access to your WSL user.
- Running `wsl-vhd down`  will unmount it and any other disks in the config file.

See `./etc/wsl-vhd.conf.test` for additional options.

## Automounting VHD's in WSL

There are two options: the `[boot]` directive in `wsl.conf` or execute with `wsl.exe`. 

### Automount with `wsl.conf`

> Note that commands executed in `[boot]` might finish before *or after* your shell loads. 
> That can be an issue if you're trying to mount `/home/` or something similar.

Add a line like this to `/etc/wsl.conf`:
```
[boot]
command = wsl-vhd up > /tmp/wsl-vhd.log 2>&1
```

If something goes wrong, a log will stored at `/tmp/wsl-vhd.log`. If you need more info, use `wsl-vhd -v` (or `-vv` for a full trace).

### Automount with `wsl.exe`

While slightly hackier, with this solution your shell loads *after* calling `wsl-vhd`.

From Windows Terminal (or your preferred), modify the command-line for the WSL distribution.

It will look something like:

```bash
wsl.exe -d Ubuntu
```

Change it to something like:

```bash
wsl.exe -d Ubuntu --exec /bin/bash -c "wsl-vhd up; exec /bin/bash --login"
```

# Uninstall

```bash
sudo rm -f $(which wsl-vhd)
```

# Changelog

## 1/3/2023

- VHD's can now be managed in a config file, `/etc/wsl-vhd.conf`.
- Added filesystem support for `ntfs`, `exfat`, `vfat`, `fat` and `msdos`
