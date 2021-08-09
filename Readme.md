# What is it about?

```
mkmm -h
```

# List the package files

```
pacman -Ql mkmm
```

# The hooks

## Hardlinks

These hooks will use hardlinks to backup and restore kernel modules. This should
be fast on modern hardware (SSD).

Read more about links: `ln(1)`, `link(2)`. Hardlinks use very limited disk space
(only inodes, no new data storage); however they are persistent and need to be
cleaned up with `mkmm bleach` after a reboot, when the modules are no longer
needed (thus, the systemd service).

```
# hyperfine --prepare='./mkmm clean && sleep 10' './mkmm save'
Benchmark #1: ./mkmm save
  Time (mean ± σ):     518.2 ms ±  82.6 ms    [User: 297.6 ms, System: 250.0 ms]
  Range (min … max):   402.1 ms … 691.3 ms    10 runs
```

```
# hyperfine --prepare='rm -rf /usr/lib/modules/5.11.13-1-ck-skylake && sleep 10' './mkmm restore'
Benchmark #1: ./mkmm restore
  Time (mean ± σ):     583.1 ms ± 112.8 ms    [User: 332.1 ms, System: 284.2 ms]
  Range (min … max):   406.6 ms … 756.0 ms    10 runs
```

```
# cd /etc/pacman.d/hooks
# ln -s /usr/share/mkmm/10-mkmm-pre.hook
# ln -s /usr/share/mkmm/10-mkmm-post.hook
# systemctl enable mkmm-bleach.service
```

## tmpfs

tmpfs(5) are temporary filesystems that live in RAM. We can use them to store
the modules but this will required permanent RAM usage until the machine
reboots (anywhere from 140 to 300MB). "Restoring" the modules is much faster
though (create a directory, mount a FS).

```
# hyperfine --prepare='./mkmm clean && sleep 10' './mkmm tsave'
Benchmark #1: ./mkmm tsave
  Time (mean ± σ):     566.6 ms ±  44.3 ms    [User: 344.0 ms, System: 286.4 ms]
  Range (min … max):   513.9 ms … 666.0 ms    10 runs
```

```
# hyperfine --prepare='./mkmm tsave && umount /usr/lib/modules/5.11.13-1-ck-skylake && rmdir /usr/lib/modules/5.11.13-1-ck-skylake && sleep 10' './mkmm trestore'
Benchmark #1: ./mkmm trestore
  Time (mean ± σ):      26.0 ms ±   6.4 ms    [User: 25.2 ms, System: 3.3 ms]
  Range (min … max):    12.9 ms …  29.4 ms    10 runs
```

```
# cd /etc/pacman.d/hooks
# ln -s /usr/share/mkmm/10-mkmm-tmpfs-pre.hook
# ln -s /usr/share/mkmm/10-mkmm-tmpfs-post.hook
# systemctl enable mkmm-bleach.service
```
