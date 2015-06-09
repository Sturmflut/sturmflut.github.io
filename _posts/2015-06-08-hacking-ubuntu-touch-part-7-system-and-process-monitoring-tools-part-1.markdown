---
layout: post
title:  "Hacking Ubuntu Touch, Part 7: System and process monitoring tools (part 1)"
date:   2015-06-08 18:00:00
categories: Ubuntu Touch
---

*NOTE: This is a continuation of the [series]({% post_url 2015-05-07-hacking-ubuntu-touch-index %}) and relies on having [Developer mode]({% post_url 2015-05-08-hacking-ubuntu-touch-part-4-developer-mode-and-ADB %}) enabled. Thanks to Oliver Grawert, Stéphane Graber, Ricardo Salveti and Selene Scriven from Canonical for their input.*


In the [last article]({% post_url 2015-05-15-hacking-ubuntu-touch-part-6-logfiles %}) I introduced you, the reader, to the different logging facilities present on current Ubuntu Touch devices and how you can query the stored messages. Well, log files are an easy way to debug easy problems, but their usefullness can be quite limited:


* The process you are looking for has to actually use a logging facility. Not every single one does, you might have noted all the missing fields in the table at the end of my previous article.

* The process has to actually log the one thing you're interested in.

* You get to have an idea about which component is at fault, or at least what the logged message could look like. Even if you're run `grep` over all logfiles, you have to know what you're looking for.


In many cases you have a clear idea about what's going wrong and there are enough log messages, e.g. let's say your phone refuses to connect to the mobile network despite the baseband being powered on and a SIM card being present. There aren't many components which are involved in mobile communications, at the end it quickly narrows down to the likes of `ofono` and `rild`, and usually the logging facilities and ofono debugging scripts give you enough information to quickly debug the problem. 

But there are a lot of other cases when you don't immediately know who is at fault, e.g. when the UI locks up or starts lagging or the battery runs out in a very short amount of time. Or you may not even know if something is happening at all, like when the phone "seems" to consume more power than usual. You might be able to find out which component is involved, but if it doesn't generate any suspicious log messages by itself, what do you do?

This article might contain some answers to these questions. I artificially limited myself to things that can be done on an unmodified phone, because you don't want/can't manipulate your phone image every time you are just looking for something, you might not have a network connection at hand to download additional software, or maybe you are assisting a friend over the phone and talking him into enabling Developer mode was already hard enough.



## ps

`ps` prints a list of all running processes, optionally augmented with information from in-kernel process descriptors and various kernel sub-systems, filtered through a list of configurable filters and modified by output options.

My standard set of options for a quick overlook is `-yfMeHl`. It selects all processes on the whole system, prints them in a "process tree", switches the output mode to "long" format, shows full field information and adds fields about memory consumption and security.

Let's look at an artificial (!) and manually filtered (!) example on my bq Aquaris E4.5 Ubuntu Edition:


{% highlight text %}
phablet@ubuntu-phablet:~$ ps -yfMeHl
LABEL                           S UID        PID  PPID  C PRI  NI   RSS    SZ WCHAN  STIME TTY          TIME CMD
unconfined                      S root         2     0  0  80   0     0     0 kthrea 07:20 ?        00:00:00 [kthreadd]
unconfined                      S root         3     2  0  80   0     0     0 run_ks 07:20 ?        00:00:02   [ksoftirqd/0]
unconfined                      S root         6     2  0 -39   -     0     0 cpu_st 07:20 ?        00:00:00   [migration/0]
unconfined                      S root        16     2  0  60 -20     0     0 rescue 07:20 ?        00:00:00   [khelper]
unconfined                      S root        17     2  0  80   0     0     0 devtmp 07:20 ?        00:00:00   [kdevtmpfs]
unconfined                      S root        18     2  0  60 -20     0     0 rescue 07:20 ?        00:00:00   [netns]
unconfined                      S root        19     2  0  60 -20     0     0 rescue 07:20 ?        00:00:00   [fs_sync]
<...>
unconfined                      S root         1     0  0  80   0  2220  1005 poll_s 07:20 ?        00:00:07 /sbin/init
unconfined                      S root       665     1  0  80   0  1108   629 poll_s 07:20 ?        00:00:00   /sbin/cgmanager --sigstop -m name=systemd
unconfined                      S root       713     1  0  80   0  1060   628 poll_s 07:20 ?        00:00:00   /sbin/cgproxy --sigstop
unconfined                      S root       802     1  0  80   0   828   620 poll_s 07:20 ?        00:00:01   upstart-local-bridge --daemon --event=android-container --path=/dev/socket/upstart-text-bridge
unconfined                      S root       803     1  0  80   0  1132   952 epoll_ 07:20 ?        00:00:00   lxc-start -n android -- /init
unconfined                      S root       895   803  0  80   0   460   169 poll_s 07:20 ?        00:00:43     /init
unconfined                      S root       925   895  0  80   0   160   142 poll_s 07:20 ?        00:00:00       /sbin/ueventd
unconfined                      S root       938   895  0  80   0   152   100 futex_ 07:20 ?        00:00:00       /sbin/upstart-property-watcher
unconfined                      S system     941   895  0  80   0   232   453 skb_re 07:20 ?        00:00:00       /system/bin/drvbd
unconfined                      S root       943   895  0  80   0   156   618 epoll_ 07:20 ?        00:00:00       /sbin/healthd
unconfined                      S root      1238     1  0  80   0  1732  1651 poll_s 07:20 ?        00:00:00   /usr/sbin/sshd -D -o PasswordAuthentication=no
unconfined                      S root     14066  1238  0  80   0  2996  2678 poll_s 18:02 ?        00:00:00     sshd: phablet [priv]                             
unconfined                      S phablet  14085 14066  0  80   0  1440  2678 poll_s 18:02 ?        00:00:02       sshd: phablet@pts/43                             
unconfined                      S phablet  14086 14085  0  80   0  1812  1619 wait   18:02 pts/43   00:00:00         -bash
unconfined                      R phablet  32231 14086  0  80   0  1140  1564 -      18:30 pts/43   00:00:00           ps -yfMeHl
<...>
unconfined                      S root      1623     1  0  80   0  3008  8877 poll_s 07:20 ?        00:00:00   lightdm
unconfined                      S root      1647  1623  0  80   0  9328 58070 epoll_ 07:20 ?        00:00:43     unity-system-compositor --disable-overlays=false --spinner=/usr/bin/unity-system-compositor-spinner --file /run/mir_socket --from-dm-fd 9 --to-dm-fd 13 --vt 1
unconfined                      S root      1723  1623  0  80   0  2500  4330 wait   07:20 ?        00:00:00     lightdm --session-child 9 16
unconfined                      S phablet   1775  1723  0  80   0  2016  1798 poll_s 07:20 ?        00:00:10       init --user
unconfined                      S phablet   2073  1775  0  80   0     0   699 poll_s 07:20 ?        00:00:00         ssh-agent -s
unconfined                      S phablet   2089  1775  0  80   0  1988  1338 epoll_ 07:20 ?        00:00:12         dbus-daemon --fork --session --address=unix:abstract=/tmp/dbus-YsvTBgQQHr
unconfined                      S phablet   2096  1775  0  80   0  3452 12993 poll_s 07:20 ?        00:00:00         /usr/lib/arm-linux-gnueabihf/url-dispatcher/url-dispatcher
unconfined                      S phablet   2100  1775  0  80   0  6864 201131 epoll_ 07:20 ?       00:00:00         nuntium
unconfined                      S phablet   2102  1775  0  80   0  1644 209983 futex_ 07:20 ?       00:00:00         usensord
/usr/bin/mediascanner-service-2.0 S phablet 2105  1775  0  80   0 13080 30530 poll_s 07:20 ?        00:00:13         mediascanner-service-2.0
unconfined                      S phablet   2115  1775  0  80   0   884  1401 poll_s 07:20 ?        00:00:00         upstart-event-bridge
unconfined                      S phablet   2127  1775  0  80   0  1932  5802 poll_s 07:20 ?        00:00:00         /usr/lib/dconf/dconf-service
unconfined                      S phablet   2131  1775  0  80   0  1004 10432 poll_s 07:20 ?        00:00:00         /usr/lib/obexd/obexd
/usr/lib/telepathy/mission-control-5 S phablet 2142 1775  0 80  0  5372  9964 poll_s 07:20 ?        00:00:02         /usr/lib/telepathy/mission-control-5
unconfined                      S phablet   2150  1775  0  69 -11  7612 76306 poll_s 07:20 ?        00:05:13         pulseaudio --start --log-target=syslog
unconfined                      S phablet   2164  1775  0  80   0   372  1402 poll_s 07:20 ?        00:00:01         upstart-dbus-bridge --daemon --session --user --bus-name session
unconfined                      S phablet   2165  1775  0  80   0   572  1560 poll_s 07:20 ?        00:00:00         upstart-file-bridge --daemon --user
<...>
unconfined                      S phablet   2669  1775  0  80   0 43832 134493 poll_s 07:20 ?       00:00:38         unity8-dash --desktop_file_hint=/usr/share/applications/unity8-dash.desktop
<...>
com.mikeasoft.podbird_Podbird_0.6 T phablet 23062 1775  0  80   0 101936 86862 signal 17:27 ?       00:00:36         /usr/lib/arm-linux-gnueabihf/qt5/bin/qmlscene $@ share/qml/podbird/podbird.qml
com.ubuntu.clock_clock_3.3.192  T phablet  25769  1775  0  80   0 35936 68146 signal 17:31 ?        00:00:04         /usr/lib/arm-linux-gnueabihf/qt5/bin/qmlscene $@ share/qml/ubuntu-clock-app.qml
com.ubuntu.music_music_2.1.857  S phablet  26119  1775  0  80   0 65204 77425 poll_s 17:32 ?        00:00:22         /usr/lib/arm-linux-gnueabihf/qt5/bin/qmlscene app/music-app.qml --url= -I ./plugins
{% endhighlight %}


Because of the "process tree" format you can immediately see that there are several hierarchies: the in-kernel threads are started by `[kthreadd]`, the Android container runs below `lxc-start -n android -- /init`, all user-space system processes are started by `/sbin/init` and all user-space user processes by an user init session started as `init --user`. You can see that apps are not started by `unity8-dash`, but `unity8-dash` tells `init --user` to start them, so there's no process hierarchy under `unity8-dash`. Also we now know that apps are normal processes and not something special, and we can see that scopes do not seem to be processes.


Now for the important columns in the output.


The first column shows the security label (`LABEL`) associated with the process. There are two possibilities: either the special value `unconfined`, which means that there are no restrictions imposed on the process, or an actual string. If there is a string set, you can find a directory of the same name in `/sys/kernel/security/apparmor/policy/profiles/` and the system has attached restrictions to this process. This is what Ubuntu Touch developers refer to as running something (like every app) in `confined` mode. We will have a closer look at confinement in a future article.


The second column shows the process state (`S`). There are seven possible values, of which only five can be actually seen:

* `D` for uninterruptible sleep. This is usually seen when the process is waiting for an I/O transaction.

* `R` for running/runnable. The process is either currently being executed on one of the CPUs, or it is not blocked by anything and is ready to be executed.

* `S` for interruptible sleep. The process is waiting for some event, e.g. a timer going off.

* `T` for stopped. There are two reasons for this: someone, e.g. the user or a job control system, stopped the process, or it is currently being traced (e.g. a debugger).

* `W` for paging. This should never be seen on a phone because this state was deprecated in the kernel 2.6 series.

* `X` for dead. This should also never be seen.

* `Z` for zombie. The process terminated, but its parent didn't go and reap it, so it still lives on.


Now for the above example: Looks pretty normal at first sight, most processes are sleeping and `ps -yfMeHl` is running (because I ran it in `phablet-shell` to get this output). But wait, what's up with all the `qmlscene` processes at the bottom? Why have they been stopped? If you run `ps -yfMeHl` on your desktop, you'll notice that state `T` hardly ever shows up. On the phone this state can be seen all the time because of the App Lifecycle, Unity8 just freezes every App which is not actively running in the foreground at the moment.


The sixth column is the CPU utilization over the lifetime of the process (`C`). This will hardly ever be anything else than "0" on the phone, because the phone sleeps all the time and no process manages to accumulate a significant amount of CPU time.


The seventh and eighth column are priority (`PRI`) and niceness (`NI`), respectively. These are used by the kernel scheduler to decide which processes run in which order and which process takes precedence if two or more "compete" for the same place. The actual meaning of both values is a kernel implementation detail and has changed a couple of times over the years, but a high priority value still means a *lower* priority, and the lower the niceness, the more favorable for the process. But why are there two values for the same thing anyways? The answer is that the current priority is calculated internally by the kernel based on different things. The niceness value on the other hand is a constant that can be defined by userspace and modifies the vallue calculated by the kernel. Without the niceness value, there wouldn't be any possibilities for the user to tell the system that he knows better or thinks different.


The ninth (`RSS`) and tenth (`SZ`) column show "Resident Set Size" and "Size", respectively. `RSS` is displayed in kilobytes and refers to the amount of actual, physical memory that's currently used by the process, so it doesn't include any pages currently swapped out. This is a major "problem" on the phone. The Aquaris E4.5 has 1 GB of RAM, but 512 MB are set up as compressed swap:

{% highlight text %}
[    0.909493] [zram_init][790] ZCompress[c030116c] ZDecompress[c0301388]
[    9.091889] zram: Initialization done!
[   39.226055] Adding 524284k swap on /dev/zram0.  Priority:-2 extents:1 across:524284k SS
{% endhighlight %}

So it is actually normal that huge portions of many processes, especially apps that haven't been used for some time, are swapped out! That's why `SZ` is more interesting, because it displays the total virtual size (all of code, data, and stack) of the process - if there wasn't an additional catch. `SZ` is in system pages, not in kilobytes. Argh. Now what's the page size on ARM?

{% highlight text %}
phablet@ubuntu-phablet:~$ getconf PAGE_SIZE
4096
{% endhighlight %}

So we have to multiply `SZ` by 4096 on this device, and we also have to keep in mind that `SZ` accounts every shared library at its full size. The sum of all virtual process sizes on my device seems to be about 10 gigabytes, so this is also less helpful. Well, nobody said memory management is easy...

Column twelve displays the process start timestamp.

Column thirteen shows the TTY associated with the process. Only interactive processes have one.

Column fourteen shows the accumulated CPU time in hours, minutes and seconds.

The last column displays the full command, including parameters. If the name is shown between brackets, it is a kernel thread.


You can sort the list by using the `--sort` parameter, like `--sort=%cpu` or `--sort=%mem`, but keep in mind that e.g. CPU utilisation is cumulative over the lifetime of the process, so you can't find out which process is the biggest CPU hog at the exact moment.

It is possible to switch from process to thread mode by adding the `-L` parameter. The output is is mostly the same, but you might get additional columns like `LWP` (thread ID within a process) and `NLWP` (number of threads in the process).




## top

`top` is the go-to tool if you want a quick, interactive overview of which process(es) and/or thread(s) on your system uses the most of a given resource, usually CPU or memory.

Let's look at a typical output:

{% highlight text %}
top - 18:06:44 up 2 days, 11:13,  2 users,  load average: 0.00, 0.01, 0.05
Tasks: 222 total,   1 running, 206 sleeping,  15 stopped,   0 zombie
%Cpu(s): 16.6 us, 15.3 sy,  0.0 ni, 67.8 id,  0.3 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:    983924 total,   942624 used,    41300 free,     8504 buffers
KiB Swap:   524284 total,   522396 used,     1888 free.    85952 cached Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                                             
 2410 phablet   20   0  359944  24404   5960 S 13.2  2.5  26:34.52 media-hub-serve                                                                                                     
 2134 phablet    9 -11  289056   3448   1980 S 10.7  0.4  25:12.35 pulseaudio                                                                                                          
  889 root      20   0     676    204    128 S  1.6  0.0   3:27.56 init                                                                                                                
 2472 phablet   20   0  607212  67276  11684 S  1.3  6.8  14:02.42 unity8                                                                                                              
21369 phablet   20   0    6228   1288    820 R  1.3  0.1   0:00.49 top                                                                                                                 
 9898 root      20   0       0      0      0 S  1.0  0.0   0:06.72 kworker/u:1                                                                                                         
14082 phablet   20   0  381068  71272   9728 S  1.0  7.2   9:53.92 qmlscene                                                                                                            
 1806 root      20   0  245820   3564   2792 S  0.6  0.4   2:54.54 unity-system-co                                                                                                     
   71 root     -95   0       0      0      0 S  0.3  0.0   2:03.30 disp_config_upd                                                                                                     
  983 root      20   0    7996    428    304 S  0.3  0.0   2:10.45 Binder_2                                                                                                            
 1959 phablet   20   0    6952   1732    800 S  0.3  0.2   0:59.98 init                                                                                                                
 2073 phablet   20   0    8956   2812    604 S  0.3  0.3   0:59.20 dbus-daemon                                                                                                         
 2315 phablet   20   0   57712   1488   1172 S  0.3  0.2   0:02.31 adbd                                                                                                                
 2572 phablet   20   0  322376  21484   6244 S  0.3  2.2   1:18.25 maliit-server                                                                                                       
{% endhighlight %}


By default the screen is sorted by CPU utilization and updated every three seconds (this is changeable with the `-d` parameter). It will run until stopped by a keypress on `q` or `Strg+c`. `top` is interactive, so you can switch between multiple pages.

The main page header shows general system statistics in the following order:

First row: current system time, uptime in days, minutes and seconds, number of active users, load average over the last one, five and fifteen minutes.

Second row: total number of processes, number of running, sleeping, stopped and zombie processes.

Third row: percentage of CPU resources spent on user processes (`us`), kernel processes (`sy`), niced processes (`ni`), being idle (`id`), waiting for I/O completion (`wa`), handling hardware (`hi`) of software (`si`) interrupts, stolen (`st`) by the hypervisor. This is cumulative over all CPUs by default.

Fourth and fifth row: total available, used and free memory/swap, memory in buffers or caches.

This already gives us a good indication of what's going on: the load average is negligible, so the system isn't overloaded with anything. The vast majority of processes is sleeping or stopped, actually only `top` is running and only two processes are consuming a bit of CPU: `media-hub-server` and `pulseaudio`, because I was listening to some music.

The memory usage looks a bit odd though: the device has 1 GB of physical memory, but 512 MB are set up as compressed swap (as mentioned before), and it looks like most of the memory is full. This is "normal" on the phone: The App Lifecycle will always keep as many apps in memory as possible. If the kernel runs out of memory, it will select the best "victim" (e.g. the app that hasn't been used for the longest time) and kills it. The system still shows the killed app in the app switcher and restarts it if selected.

The process list on the main page is sorted by top CPU usage and shows the following columns:

`PID`, `USER`, `PRI` and `NI` are identical to the columns of the same name shown by `ps`.

`VIRT`, `RES` and `SHR` display the virtual, resident and shared memory size of the process in kilobytes. `VIRT` is the total size of all sections, regardless if in memory or swapped out. `RES` is identical to the `RSS` column displayed by `ps`. `SHR` is the amount of shared memory available to the process, this is a "potential" value, so it doesn't say that the memory is actually shared.

`S` is identical to the `S` column shown by `ps`.

`%CPU` and `%MEM` display the percentage of CPU and physical RAM used by a process, but in contrast to `ps` the values are updated between iterations.

There are several possibilities to change the content of the process table:

* You can change which fields are displayed in which order by pressing the `f` key. It leads to an interactive screen.

* The `c` key extends the `COMMAND` column to full length. This is usually necessary because the field is cut off.

* The `o` key prompts for a filter criteria. See the man page for an introduction on how to write a filter rule.

* The `u` key prompts for a username or user ID, if entered only processes owned by this user will be shown.

* The `V` key activates a "process tree" display like shown with the `ps` command before. Obviously you will need a wide terminal for this.

* By default the sort field is `%CPU`. You can switch between the currently displayed columns with the `<` (left) and `>` (right) keys.

The man page contains many more key bindings and options.



## vmstat

`vmstat` initially only displayed information about the kernel virtual memory subsystem and was later extended to display additional system information. It can be run in two modes: when you just call `vmstat` without an interval, it ouputs values since the last reboot. When you specific an interval, like `vmstat 1`, it first outputs the values since the last reboot, but afterwards continues to print a delta every n seconds.


{% highlight text %}
phablet@ubuntu-phablet:~$ vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 358224  11228 108568    0    0    95    38  277  376  3  2 94  0  0
 0  0      0 358240  11228 108572    0    0     0     0   67  114  0  0 100  0  0
 0  0      0 358240  11228 108572    0    0     0     0   56  101  0  0 100  0  0
 0  0      0 358240  11228 108572    0    0     0     8   59  100  0  1 99  0  0
 0  0      0 358232  11228 108572    0    0     0     0   68  148  0  5 95  0  0
 1  0      0 357696  11244 108556    0    0     8   648  967 2024 20 12 67  1  0
 0  0      0 357696  11284 108580    0    0     0   184 2743 3523 13  3 84  1  0
 0  0      0 357688  11284 108580    0    0     0   100 2704 3442 15  4 81  0  0
 0  0      0 357184  11416 108580    0    0   124    88 2398 3208 26  8 66  0  0
 0  0      0 357648  11416 108580    0    0     0     0 2853 4090  8 11 80  0  0
 1  0      0 374760  11428 108584    0    0     0   144 2803 3789 27 18 55  0  0
 1  0      0 372644  11992 108792    0    0   776     0 3418 4795 29  8 63  1  0
 0  0      0 373892  11992 108796    0    0     0     0 2443 3565 13 10 77  0  0
 0  0      0 373892  11992 108796    0    0     0     4  628  854  1  0 99  0  0
{% endhighlight %}

The default output is optimized for terminals 80 columns wide and shows 17 different values:

`r` and `b` is the number of processes in the "running" and "uninterruptible sleep" states.

`swpd` is the amount of used swap, `free` the amount of free memory, `buff` the amount of memory used for I/O buffers and `cache` the amount of memory used as read caches.

`si` and `so` is the amount of memory swapped in from/out to disk, per second.

`bi` and `bo` is the amount of blocks read/written to block devices, per second.

`in` and `cs` is the number of interrupts and context switches, per second. The interrupt counter includes the clock and peripherials, so this number wont't go down even if the phone goes to sleep.

`us`, `sy`, `id`, `wa` and `st` are identical to the colums of the same name displayed by `top`.






If you know better and/or something has changed, please do find me on the Freenode IRC or on Launchpad.net and get in contact!
