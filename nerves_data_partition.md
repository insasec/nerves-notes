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
iex(2)> cmd "ls -alhd /data"
lrwxrwxrwx    1 root     root           4 Jul 23 14:33 /data -> root
0
iex(3)> cmd "ls -alh /root" 
drwxr-xr-x    3 root     root        3.4K Jul 25  2020 .cache
drwxr-xr-x    3 root     root        3.4K Jul 25  2020 nerves_ssh
-rw-r--r--    1 root     root           0 Aug 16 08:57 .nerves_time
drwxr-xr-x   19 1000     1000         257 Jul 23 14:33 ..
drwxr-xr-x    4 root     root        4.0K Jul 25  2020 .
0
iex(4)> File.write("/root/user.data", "Hello World")
:ok
```


> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQ3MDk3NDk1MV19
-->