# wsl-vhd

Create, manage and automount VHD's on boot.

The blogpost which inspired this project: <https://kmmiles.github.io/wsl/vhd/linux/2022/12/20/wsl-vhd.html>

## Windows Requirements

 - Windows 11 21H2 or higher.
 - WSL installed from the Microsoft Store: <https://devblogs.microsoft.com/commandline/a-preview-of-wsl-in-the-microsoft-store-is-now-available/>

## Linux requirements

### Minimum

Debian/Ubuntu: `sudo apt install util-linux qemu-utils`

Redhat/Fedora: `sudo dnf install util-linux qemu-img`

### With additional filesystems

Adds support for `btrfs`, `ntfs`, `exfat` and `vfat`/`fat`/`msdos`: 

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
curl -Ls 'https://raw.githubusercontent.com/kmmiles/wsl-vhd/main/wsl-vhd' | \
  sudo tee /usr/local/bin/wsl-vhd > /dev/null && sudo chmod +x /usr/local/bin/wsl-vhd
```

Or check out the project and symlink it:

```bash
cd ~
git clone "https://github.com/kmmiles/wsl-vhd.git"
cd wsl-vhd
sudo ln -sf "$(pwd)/wsl-vhd" /usr/local/bin/wsl-vhd
```

> `wsl-vhd` just needs to exist somewhere that's included in both your and the `root` users `$PATH`.

## Configuration

Create and add the following to `/etc/wsl-vhd.conf`:

```
[global]
uid=1000
gid=1000

[C:\wsl-vhd\test-ext4.vhdx]
fstype=ext4
size_in_mb=100
```

Now run `wsl-vhd up`.

- `C:\wsl-vhd\test-ext4.vhdx` will be created, formatted and mounted at `/mnt/wsl/test-ext4`.
- The mountpoint will be given read/write access to your WSL user.
- Running `wsl-vhd down`  will unmount it and any other disks in the config file.

See `./etc/wsl-vhd.conf.test` for additional options.

## Run on boot

> When run on boot, `wsl-vhd` is executed as `root`. 

Add a line like this to `/etc/wsl.conf`:

```
[boot]
command = wsl-vhd up > /tmp/wsl-vhd.log 2>&1
```

If something goes wrong, a log will stored at `/tmp/wsl-vhd.log`. If you need more info, replace `-v` with `-vv` (or `-vvv` for the verbosiest among us).

# Additional usage

Run `wsl-vhd -h` to view additional commands. 

# Changelog

## 1/3/2023

- VHD's can now be managed in a config file, `/etc/wsl-vhd.conf`.
- Added ability to specify `partition` in configuration.
- Added filesystem support for `ntfs`, `exfat`, `vfat`, `fat` and `msdos`
