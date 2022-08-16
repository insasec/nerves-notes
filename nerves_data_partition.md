# `/data` on a Nerves system
A Nerves application should only store user data under `/data` (see [Where can persistent data be stored?](https://hexdocs.pm/nerves/faq.html#where-can-persistent-data-be-stored)). On first glance one might assume that `/data` is a mount point for the application data partition. However checking on a Nerves system yields:

```elixir
iex(1)> cmd "ls -alhd /data"
lrwxrwxrwx    1 root     root           4 Jul 23 14:33 /data -> root
0
```

So `/data` is a symbolic link to `/root` which is `root`'s home directory.

```elixir
iex(3)> cmd "ls -alh /root" 
drwxr-xr-x    3 root     root        3.4K Jul 25  2020 .cache
drwxr-xr-x    3 root     root        3.4K Jul 25  2020 nerves_ssh
-rw-r--r--    1 root     root           0 Aug 16 08:57 .nerves_time
drwxr-xr-x   19 1000     1000         257 Jul 23 14:33 ..
drwxr-xr-x    4 root     root        4.0K Jul 25  2020 .
0
``` 

Checking the mountpoints one can see that `/root` is a [`f2fs` Filesystem](https://en.wikipedia.org/wiki/F2FS) mounted from `/dev/mmcblk0p4`. So this is partition 4 (`p4`) on the first mmc block device (`mmcblk0`). This is also in line with Nerves documentation on [Partitions](https://hexdocs.pm/nerves/advanced-configuration.html#partitions):

```elixir
iex(1)> cmd "mount"
/dev/mmcblk0p3 on / type squashfs (ro,relatime)
devtmpfs on /dev type devtmpfs (rw,nosuid,noexec,relatime,size=1024k,nr_inodes=57279,mode=755)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
tmpfs on /tmp type tmpfs (rw,nosuid,nodev,noexec,relatime,size=50844k)
tmpfs on /run type tmpfs (rw,nosuid,nodev,noexec,relatime,size=25424k,mode=755)
/dev/mmcblk0p1 on /mnt/boot type vfat (ro,nosuid,nodev,noexec,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro)
/dev/mmcblk0p4 on /root type f2fs (rw,nodev,relatime,lazytime,background_gc=on,discard,no_heap,inline_data,inline_dentry,flush_merge,extent_cache,mode=adaptive,active_logs=6,alloc_mode=default,fsync_mode=posix)
tmpfs on /sys/fs/cgroup type tmpfs (rw,nosuid,nodev,noexec,relatime,size=1024k,mode=755)
cpu on /sys/fs/cgroup/cpu type cgroup (rw,nosuid,nodev,noexec,relatime,cpu)
memory on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
none on /sys/kernel/config type configfs (rw,relatime)
0
```

As expected you can of course write to `/root` or `/data`:
```elixir
iex(1)> File.write("/root/user.data", "Hello World")
:ok
```

If you plugin the card into a PC you are also able to read/write on this partition.

> :warning: **If you plug the SD Card into a PC or Mac you won't see the content!** `f2fs` is only supported on Linux systems at the moment.

```
udos@tuxbook:~$ mount
[...]
/dev/mmcblk1p1 on /media/udos/BOOT type vfat (rw,nosuid,nodev,relatime,uid=1000,gid=1000,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,showexec,utf8,flush,errors=remount-ro,uhelper=udisks2)
/dev/mmcblk1p2 on /media/udos/disk type squashfs (ro,nosuid,nodev,relatime,errors=continue,uhelper=udisks2)
/dev/mmcblk1p3 on /media/udos/disk1 type squashfs (ro,nosuid,nodev,relatime,errors=continue,uhelper=udisks2)
/dev/mmcblk1p4 on /media/udos/553921f6-9b97-4e0e-a08f-4b4ebb7f70e2 type f2fs (rw,nosuid,nodev,relatime,lazytime,background_gc=on,discard,no_heap,user_xattr,inline_xattr,acl,inline_data,inline_dentry,flush_merge,extent_cache,mode=adaptive,active_logs=6,alloc_mode=default,checkpoint_merge,fsync_mode=posix,discard_unit=block,uhelper=udisks2)

udos@tuxbook:~$ lsblk /dev/mmcblk1
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
mmcblk1     179:0    0 29,7G  0 disk 
├─mmcblk1p1 179:1    0   14M  0 part /media/udos/BOOT
├─mmcblk1p2 179:2    0  140M  0 part /media/udos/disk
├─mmcblk1p3 179:3    0  140M  0 part /media/udos/disk1
└─mmcblk1p4 179:4    0 29,4G  0 part /media/udos/553921f6-9b97-4e0e-a08f-4b4ebb7f70e2
```

On a Linux system you should be able to access the `/root`/`/data` partition and verify the content we've written from Nerves above:

```
udos@tuxbook:~$ cd /media/udos/553921f6-9b97-4e0e-a08f-4b4ebb7f70e2/
udos@tuxbook:/media/udos/553921f6-9b97-4e0e-a08f-4b4ebb7f70e2$ ls
nerves_ssh  user.data
udos@tuxbook:/media/udos/553921f6-9b97-4e0e-a08f-4b4ebb7f70e2$ cat user.data 
Hello World
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbODY3MTMwNjUxLDcwMDAxNTAwMywxMjgwMD
I0MDA3LDE0NzA5NzQ5NTFdfQ==
-->