# ZFS-on-WSL

ZFS? In my WSL? ... It's more likely than you think.

This is a set of scripts and methods for building a WSL2 kernel and the corresponding userspace utilities necessary to support ZFS within WSL.

This does work, but it's quick and nasty so this is your obligatory warning not to rely on it for anything production-grade.

Self-building by running `./build_wsl_kernel.sh` directly inside of an Ubuntu WSL environment.

### Procedure

Open your WSL Ubuntu instance.

Clone this repo and go to the repo root directory.

Run:
```
sudo ./build_wsl_kernel.sh
```

Stop the WSL2 VM:
```bat
wsl --shutdown
```

Edit the `.wslconfig` file in your Windows user's home directory to point to the downloaded kernel:
```ini
[wsl2]
kernel=C:\\ZFSonWSL\\bzImage-new
localhostForwarding=true
swap=0
```

Start up WSL again by opening a new WSL session and check that our custom kernel is being used:
```
$ uname -a
Linux NPC 5.15.137-penguins-rule #1 SMP Sun Oct 29 11:27:47 CET 2023 x86_64 x86_64 x86_64 GNU/Linux
```

Download the kernel and ZFS `.tgz` files and extract them. The commands below will put everything in `/usr/src` as we will need:
```
cd /tmp/kbuild/

sudo tar -zxvf linux-5.15.137-penguins-rule.tgz -C /

sudo tar -zxvf zfs-2.2.0-for-5.15.137-penguins-rule.tgz -C /
```

Change directory into the kernel dir:
```
cd /usr/src/linux-5.15.137-penguins-rule/
```

Install our kernel modules:
```
sudo make modules_install
```

Change directory into the ZFS dir:
```
cd /usr/src/zfs-2.2.0-for-5.15.137-penguins-rule
```

Install our ZFS userspace utilities:
```
sudo make install
```

The command above should create the relevant files and links in `/lib/modules` for the built kernel modules.

You should now be able to insert the ZFS modules into the kernel without any issues:
```
sudo modprobe zfs
```

And you should be able to use the ZFS userspace utilities:
```
sudo zpool status
```

Now you can create ZFS pools within WSL2 by passing raw disks through to the WSL2 VM, [as described by the Microsoft Docs for WSL](https://docs.microsoft.com/en-us/windows/wsl/wsl2-mount-disk).
