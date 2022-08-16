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

Checking the mountpoints one can
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


iex(4)> File.write("/root/user.data", "Hello World")
:ok
```

udos@tuxbook:~$ cd /media/udos/553921f6-9b97-4e0e-a08f-4b4ebb7f70e2/
udos@tuxbook:/media/udos/553921f6-9b97-4e0e-a08f-4b4ebb7f70e2$ ls
nerves_ssh  user.data
udos@tuxbook:/media/udos/553921f6-9b97-4e0e-a08f-4b4ebb7f70e2$ cat user.data 
Hello Worldudos@tuxbook:/media/udos/553921f6-9b97-4e0e-a08f-4b4ebb7f70e2$ 


udos@tuxbook:/media/udos/553921f6-9b97-4e0e-a08f-4b4ebb7f70e2$ mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
udev on /dev type devtmpfs (rw,nosuid,relatime,size=16363632k,nr_inodes=4090908,mode=755,inode64)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,nodev,noexec,relatime,size=3279788k,mode=755,inode64)
/dev/sda5 on / type ext4 (rw,relatime,errors=remount-ro)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev,inode64)
tmpfs on /run/lock type tmpfs (rw,nosuid,nodev,noexec,relatime,size=5120k,inode64)
cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursiveprot)
pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime)
bpf on /sys/fs/bpf type bpf (rw,nosuid,nodev,noexec,relatime,mode=700)
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=29,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=18028)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,pagesize=2M)
mqueue on /dev/mqueue type mqueue (rw,nosuid,nodev,noexec,relatime)
debugfs on /sys/kernel/debug type debugfs (rw,nosuid,nodev,noexec,relatime)
tracefs on /sys/kernel/tracing type tracefs (rw,nosuid,nodev,noexec,relatime)
fusectl on /sys/fs/fuse/connections type fusectl (rw,nosuid,nodev,noexec,relatime)
configfs on /sys/kernel/config type configfs (rw,nosuid,nodev,noexec,relatime)
none on /run/credentials/systemd-sysusers.service type ramfs (ro,nosuid,nodev,noexec,relatime,mode=700)
/var/lib/snapd/snaps/bare_5.snap on /snap/bare/5 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/core_13308.snap on /snap/core/13308 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/core_13425.snap on /snap/core/13425 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/core18_2409.snap on /snap/core18/2409 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/core18_2538.snap on /snap/core18/2538 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/core20_1587.snap on /snap/core20/1587 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/firefox_1635.snap on /snap/firefox/1635 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/firefox_1670.snap on /snap/firefox/1670 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/gnome-3-28-1804_145.snap on /snap/gnome-3-28-1804/145 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/gnome-3-28-1804_161.snap on /snap/gnome-3-28-1804/161 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/gnome-3-38-2004_106.snap on /snap/gnome-3-38-2004/106 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/gnome-3-38-2004_112.snap on /snap/gnome-3-38-2004/112 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/gtk-common-themes_1534.snap on /snap/gtk-common-themes/1534 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/gtk-common-themes_1535.snap on /snap/gtk-common-themes/1535 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/intellij-idea-community_372.snap on /snap/intellij-idea-community/372 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/intellij-idea-community_381.snap on /snap/intellij-idea-community/381 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/snap-store_558.snap on /snap/snap-store/558 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/snap-store_582.snap on /snap/snap-store/582 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/snapd_16010.snap on /snap/snapd/16010 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/snapd_16292.snap on /snap/snapd/16292 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/var/lib/snapd/snaps/snapd-desktop-integration_14.snap on /snap/snapd-desktop-integration/14 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/dev/sda1 on /boot/efi type vfat (rw,relatime,fmask=0077,dmask=0077,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro)
tmpfs on /run/snapd/ns type tmpfs (rw,nosuid,nodev,noexec,relatime,size=3279788k,mode=755,inode64)
nsfs on /run/snapd/ns/snapd-desktop-integration.mnt type nsfs (rw)
tmpfs on /run/user/1000 type tmpfs (rw,nosuid,nodev,relatime,size=3279784k,nr_inodes=819946,mode=700,uid=1000,gid=1000,inode64)
gvfsd-fuse on /run/user/1000/gvfs type fuse.gvfsd-fuse (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000)
portal on /run/user/1000/doc type fuse.portal (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000)
/var/lib/snapd/snaps/core20_1593.snap on /snap/core20/1593 type squashfs (ro,nodev,relatime,errors=continue,x-gdu.hide)
/dev/mmcblk1p1 on /media/udos/BOOT type vfat (rw,nosuid,nodev,relatime,uid=1000,gid=1000,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,showexec,utf8,flush,errors=remount-ro,uhelper=udisks2)
/dev/mmcblk1p2 on /media/udos/disk type squashfs (ro,nosuid,nodev,relatime,errors=continue,uhelper=udisks2)
/dev/mmcblk1p3 on /media/udos/disk1 type squashfs (ro,nosuid,nodev,relatime,errors=continue,uhelper=udisks2)
/dev/mmcblk1p4 on /media/udos/553921f6-9b97-4e0e-a08f-4b4ebb7f70e2 type f2fs (rw,nosuid,nodev,relatime,lazytime,background_gc=on,discard,no_heap,user_xattr,inline_xattr,acl,inline_data,inline_dentry,flush_merge,extent_cache,mode=adaptive,active_logs=6,alloc_mode=default,checkpoint_merge,fsync_mode=posix,discard_unit=block,uhelper=udisks2)
udos@tuxbook:/media/udos/553921f6-9b97-4e0e-a08f-4b4ebb7f70e2$ lsblk /dev/mmcblk1
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
mmcblk1     179:0    0 29,7G  0 disk 
├─mmcblk1p1 179:1    0   14M  0 part /media/udos/BOOT
├─mmcblk1p2 179:2    0  140M  0 part /media/udos/disk
├─mmcblk1p3 179:3    0  140M  0 part /media/udos/disk1
└─mmcblk1p4 179:4    0 29,4G  0 part /media/udos/553921f6-9b97-4e0e-a08f-4b4ebb7f70e2
udos@tuxbook:/media/udos/553921f6-9b97-4e0e-a08f-4b4ebb7f70e2$ 


> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzY0OTc5NjMyLDEyODAwMjQwMDcsMTQ3MD
k3NDk1MV19
-->