## Non-Uniform Access Memory (NUMA) Hardware

For the majority of use cases, running q on a NUMA system can cause a number of operational issues, including high system-process usage and poor performance. Hence for these systems, you should disable NUMA and set an interleave memory policy. Q is unaware of whether NUMA is enabled or not.

If possible, disable NUMA in bios. Otherwise use the technique below.

To fully disable NUMA and set an interleave memory policy, start q with the `numactl` command as follows
```bash
$ numactl --interleave=all q
```
_and_ disable zone-reclaim in the proc settings as follows
```bash
$ echo 0 > /proc/sys/vm/zone_reclaim_mode
```
<i class="fa fa-hand-o-right"></i> [The MySQL “swap insanity” problem and the effects of NUMA](http://jcole.us/blog/archives/2010/09/28/mysql-swap-insanity-and-the-numa-architecture/)
Although the post is about the impact on MySQL, the issues are the same for other databases such as q.

To find out whether NUMA is enabled in your bios, use
```bash
$ dmesg | grep -i numa
```
And to see if NUMA is enabled on a process basis
```bash
$ numactl -s
```


## Huge Pages and Transparent Huge Pages (THP)

A number of customers have been impacted by bugs in the Linux kernel with respect to Transparent Huge Pages. These issues manifest themselves as process crashes, stalls at 100% CPU usage, and sporadic performance degradation. Until further notice, we strongly recommend disabling THP on Linux systems that run q. 

Other database vendors are also reporting similar issues with THP.

Note that disabling Transparent Huge Pages isn’t possible via `sysctl(8)`. Rather, it requires manually echoing settings into /sys/kernel at or after boot. In /etc/rc.local or by hand:
```bash
if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
  echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi

if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
  echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi
```
Redhat users may need a slightly different path
```bash
$ echo never >/sys/kernel/mm/redhat_transparent_hugepage/enabled
```
Another possibility to configure this is via grub
```
transparent_hugepage=never
```
Q must be restarted to pick up the new setting.


## Monitoring free disk space

In addition to monitoring free disk space for your usual partitions which you write to, ensure you also monitor free space of /tmp on Unix, since q uses this area for capturing the output from system commands, such as `system "ls"`.


## Compression

If you find that q is seg faulting (crashing) when accessing compressed files, try increasing the Linux kernel parameter `vm.max_map_count`. As root
```bash
$ sysctl vm.max_map_count=16777216
```
and/or make a suitable change for this parameter more permanent through /etc/sysctl.conf. As root
```bash
$ echo "vm.max_map_count = 16777216" | tee -a /etc/sysctl.conf
$ sysctl -p
```
You can check current settings with
```bash
$ more /proc/sys/vm/max_map_count
```
Assuming you are using 128kB logical size blocks for your compressed files, a general guide is, at a minimum, set `max_map_count` to 1 map per 128kB of memory, or 65530, whichever is higher.

If you are encountering a SIGBUS error, please check that the size of /dev/shm is large enough to accommodate the decompressed data. Typically, you should set the size of /dev/shm to be at least as large as a fully decompressed HDB partition.


## Timekeeping

Timekeeping on production servers is a complicated topic. These are just a few notes which can help.

If you are using any of local time functions `.z.(TPNZD)` q will use the `localtime(3)` system function to determine time offset from GMT. In some setups (GNU libc) this can cause excessive system calls to /etc/localtime.  
<i class="fa fa-hand-o-right"></i> [chemie.fu-berlin.de](http://www.chemie.fu-berlin.de/chemnet/use/info/libc/libc_17.html#SEC301), [stackoverflow.com](http://stackoverflow.com/questions/4554271/how-to-avoid-excessive-stat-etc-localtime-calls-in-strftime-on-linux/4554302#4554302)

Setting TZ environment helps this:
```bash
$ export TZ=America/New_York
```
or from q
```q
q)setenv[`TZ;"Europe/London"]
```
One more way of getting excessive system calls when using `.z.(pt…)` is to have a slow clock source configured on your OS. Modern Linux distributions provide very low overhead functionality for getting current time. Use `tsc` clocksource to activate this.
```bash
$ echo tsc >/sys/devices/system/clocksource/clocksource0/current_clocksource
# list available clocksource on the system
$ cat /sys/devices/system/clocksource/clocksource*/available_clocksource
```
If you are using PTP for timekeeping, your PTP hardware vendor might provide their own implementation of time. Check that those utilize VDSO mechanism for exposing time to user space.
