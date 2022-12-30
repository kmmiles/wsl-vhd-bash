# wsl-vhd

Automagically create/format/mount VHD disk images in WSL on boot.

<https://kmmiles.github.io/wsl/vhd/linux/2022/12/20/wsl-vhd.html>

## Windows Requirements

 - Windows 11 21H2 or higher.
 - WSL installed from the Microsoft Store: <https://devblogs.microsoft.com/commandline/a-preview-of-wsl-in-the-microsoft-store-is-now-available/>

## Linux requirements

Debian/Ubuntu: `sudo apt install qemu-utils ntfs-3g util-linux`

Redhat/Fedora: `sudo dnf install qemu-img ntfsprogs util-linux`

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

## Manual usage

> Invoke as a normal user (don't use `sudo`).

```
Usage: wsl-vhd [GLOBAL-OPTIONS] <COMMAND>

VHD management for WSL.

GLOBAL OPTIONS

  -h  This message
  -v  Increase verbosity. `-vv` for debug, `-vvv` for extended debug.

COMMANDS

  use           Mount VHD's, creating and formatting if necessary
  create        Create new VHD without formatting
  attach        Attach VHD without mounting (aka bare mount)
  mount         Mount VHD
  format        Format VHD
  unmount       Unmount VHD (or all by default)
  compact       Compact VHD (aka sparsify)

Pass `-h` to any command to view usage e.g. `wsl-vhd use -h`
```

## Run on boot

> When run on boot, `wsl-vhd` is executed as `root`. 

Add a line like this to `/etc/wsl.conf`:

```
[boot]
command = wsl-vhd -v use -u myuser /mnt/c/wsl-vhd/code.vhdx ext4 10000 /tmp/wsl-vhd.log 2>&1
```
Replacing `myuser` with your username and any other adjustments. Then shutdown WSL with `wsl --shutdown` and restart.
Multiple images can be handled by tacking on more arguments e.g.

```
[boot]
command = wsl-vhd -v use -u myuser \
 /mnt/c/wsl-vhd/code.vhdx ext4 10000 \
 /mnt/c/wsl-vhd/code2.vhdx btrfs 20000 \
 /mnt/c/wsl-vhd/code3.vhdx ntfs 30000 > \
 /tmp/wsl-vhd.log 2>&1
```

If something goes wrong, a log will stored at `/tmp/wsl-vhd.log`. If you need more info, replace `-v` with `-vv` (or `-vvv` for the verbosiest among us).
