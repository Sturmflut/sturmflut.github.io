---
layout: post
title:  "Hacking Ubuntu Touch, Part 8: System and process monitoring tools (part 2)"
date:   2015-11-29 18:30:00
categories: Ubuntu Touch
---

*NOTE: This is a continuation of the [series]({% post_url 2015-05-07-hacking-ubuntu-touch-index %}) and relies on having [Developer mode]({% post_url 2015-05-08-hacking-ubuntu-touch-part-4-developer-mode-and-ADB %}) enabled.*

*NOTE: Thanks to Colin King from Canonical for all the help, and to Thomas Voss from Canonical for sheding some light on the GPS issue mentioned in the `eventstat` section.*


In the [first part]({% post_url 2015-06-08-hacking-ubuntu-touch-part-7-system-and-process-monitoring-tools-part-1 %}) of this article we had a look at the "usual" suspects for system and process monitoring: `ps`, `top` and `vmstat`. In this second part we will look at some special tools developed to find runaway processes and other power hogs on Ubuntu phones.

All tools mentioned on this page can be found on Colin King's [website][canonical-colin] and are available in the [ppa:colin-king/white][colin-ppa] PPA. The easiest way to get them onto your device is to download the pre-built armhf deb packages from the [PPA file mirror][colin-ppa-mirror], push them to the device and use `dpkg -x` to extract the packages to a temporary directory.


## cpustat

`cpustat` displays the cpu utilization of all currently running tasks. It takes two optional arguments: the delay between iterations (1 second by default), and the number of iterations to run (infinite by default).


{% highlight text %}
phablet@ubuntu-phablet:/tmp$ ./cpustat 
  %CPU   %USR   %SYS   PID S  CPU   Time Task
  3.00   0.00   3.00 27007 R    0  0.15s bin/cpustat
  1.00   0.00   1.00  2066 S    0  7.12m upstart
  1.00   0.00   1.00  2399 S    0  5.43s adbd

  %CPU   %USR   %SYS   PID S  CPU   Time Task
  3.03   0.00   3.03 27007 R    0  0.18s bin/cpustat
  1.01   0.00   1.01 22621 S    0 42.60s telegram
  1.01   0.00   1.01 26203 S    0 26.75m unity8
  1.01   1.01   0.00 26225 S    1  3.94m maliit-server

  %CPU   %USR   %SYS   PID S  CPU   Time Task
  3.96   0.00   3.96 27007 R    0  0.22s bin/cpustat
  1.98   0.00   1.98 26203 S    0 26.76m unity8
  0.99   0.00   0.99 25233 S    0  0.32s sshd: phablet@pts/51
  0.99   0.00   0.99  2399 S    0  5.44s adbd
  0.99   0.99   0.00  2509 S    0 21.26s /usr/lib/arm-linux-gnueabihf/indicator-datetime/indicator-datetime-service
  0.99   0.00   0.99 19893 S    0  0.90s [kworker/0:2]
  0.99   0.00   0.99    71 S    0  5.55m [disp_config_upd]
{% endhighlight %}

The man page contains quite a list of command line options which modify output. I found the following very useful:

* `-i` ignores `cpustat` itself in the output. This should probably be on by default?

* `-D` shows the distribution of CPU utilisation by task and by CPU once the command terminates.

* `-l` shows full command information.

* `-r` writes to a CSV file for later analysis.

* `-S` adds timestamps to the output.

* `-t` ignores processes which used less CPU than a given threshold in the last interval.

* `-x` shows the average CPU load over the last 1, 5 and 10 minutes, the average CPU frequency over all CPUs and the number of online CPUs.



## eventstat

This tool periodically reads the content of the kernel event statistics file, `/proc/timer_stats`, and filters/beautifies it. The output gives a good indication about which processes and threads use kernel timers to be woken up at specific points in time, and how often that happens. An excessive number of timer events will most likely keep the system unnecessarily busy and keep it from going to sleep, thus increasing power consumption.

The syntax is mostly the same as `cpustat`: `eventstat` takes `delay` and `count` as optional arguments, the default is to update the output every second until `Strg+c` is pressed.

Now for the output:


{% highlight text %}
root@ubuntu-phablet:/tmp# ./eventstat 
 Event/s PID   Task            Init Function             Callback
   44.90     0 [swapper/0]     hrtimer_start_range_ns    tick_sched_timer
   10.20  3188 ubuntu-location hrtimer_start_range_ns    hrtimer_wakeup
    6.12 23678 [kworker/0:0]   queue_delayed_work        delayed_work_timer_fn
    3.06    60 [btif_rxd]      wait_for_common           process_timeout
    2.04 26377 QSGRenderThread hrtimer_start_range_ns    posix_timer_fn
    2.04     0 [swapper/0]     __pm_wakeup_event         pm_wakeup_timer_fn
    2.04 22679 QSGRenderThread hrtimer_start_range_ns    posix_timer_fn
    2.04 26432 QSGRenderThread hrtimer_start_range_ns    posix_timer_fn
    2.04  1228 QSGRenderThread hrtimer_start_range_ns    posix_timer_fn
    2.04  2027 Mir/Comp        hrtimer_start_range_ns    posix_timer_fn
    2.04 15982 [tx_thread]     schedule_timeout_uninterruptible process_timeout
    2.04 26225 maliit-server   hrtimer_start_range_ns    posix_timer_fn
    2.04 26240 ubuntu-push-cli hrtimer_start_range_ns    hrtimer_wakeup
    1.02    65 [esd_recovery_] schedule_timeout_interruptible process_timeout
    1.02    55 [hang_detect]   start_bandwidth_timer     sched_rt_period_timer
    1.02     0 [swapper/0]     add_timer                 wmt_cal_stats
    1.02 26249 ubuntu-push-cli hrtimer_start_range_ns    hrtimer_wakeup
    1.02 25998 [kworker/u:2]   hrtimer_start             charger_hv_detect_sw_workaround
    1.02 25233 sshd            sk_reset_timer            tcp_write_timer
    1.02 23678 [kworker/0:0]   schedule_timeout_uninterruptible process_timeout
    1.02 22651 QQmlThread      hrtimer_start_range_ns    hrtimer_wakeup
    1.02  1922 unity-system-co hrtimer_start_range_ns    hrtimer_wakeup
    1.02 26206 MirServerThread hrtimer_start_range_ns    hrtimer_wakeup
    1.02     0 [swapper/0]     mlog_timer_handler        mlog_timer_handler
92 Total events, 93.88 events/sec (kernel: 64.29, userspace: 29.59)
{% endhighlight %}

The first column is the number of events per task per second, the second is *the thread ID (`LWP` column in ps)*, the third is the task name, column four is the kernel function that was used to initialize the timer and column five the kernel function called on expiry.

In the example you can see that `ubuntu-location-serviced` (thread `2976`) is called ten times per second, even though I had the Location detection completely disabled on my bq Aquaris E4.5. I talked about this with Thomas Voss at Canonical, and apparently the Android Hardware Abstraction Layer for the GPS in the Aquaris E4.5 "ticks" at a constant rate of 10 Hertz, even if the GPS is deactivated completely. Luckily this doesn't prevent the CPU from going into deep sleep, so the increase in power consumption is negligible.

`eventstat` again has a lot of command line options, the following ones seem the most interesting to me:

* `-r` again writes the output to a CSV file for later analysis.

* `-l` switches the process name field to "long" format.

* `-k` shows only kernel threads, `-u` only user-space processes.

* `-w` adds a timestamp to the output.



## fnotifystat

Most file system activity is invisible to the user, but it can have a serious impact on system performance and power consumption. This is where `fnotifystat` comes into play: it uses fnotify to dump all filesystem activity in a given period of time. `fnotifystat` again takes two optional arguments, the delay and the count. The defaults are a delay of one second between iterations and an infinite number of iterations, but unless you specify the `-f` option output will be suppressed if there was no activity in the last iteration (e.g. when the device is sleeping).


{% highlight text %}
root@ubuntu-phablet:/tmp# ./fnotifystat -f
Total   Open  Close   Read  Write   PID  Process         Pathname
  3.0    1.0    1.0    0.0    1.0    944 powerd          /sys/power/wake_lock
  2.0    0.0    0.0    0.0    2.0    807 rsyslogd        /var/log/syslog
  1.0    0.0    0.0    1.0    0.0    807 rsyslogd        /proc/kmsg
  1.0    1.0    0.0    0.0    0.0    944 powerd          /sys/power/state
{% endhighlight %}


The output displays how many open/close/read/write operations a process requested on a specific file *on average over the sampling period*. Keep this in mind if your delay is not set to one second!

Out of the long list of command line options I ended up using the following more often:

* `-f` forces output after every iteration even if there was no activity.

* `-D` switches to "device mode" and displays the minor/major device number instead of the filename.

* `-I` is like `-D`, but includes file system inodes.

* `-i` and `-x` can be used to whitelist/blacklist paths, instead of monitoring all file systems.

* `-m` merges multiple events on the same file by the same process into a single line, instead of writing to the output every time the kernel sends a notification.

* `-p` limits monitoring to the specified process names/IDs.

There doesn't currently seem to be an option to write the output to a CSV file.





## forkstat

Two of the most important system calls of any UNIX-like system are `fork`, `exec` and `exit`, used to create child processes. On Linux `fork` has been mostly replaced by `clone`. While the implementation of these system calls is usually highly optimized, a high number of spawned and killed child processes and threads can have negative effects on system performance and power consumption.

`forkstat` monitors the system and displays every call to `fork`, `clone`, `exec` and `exit`. It also tells you how long a process/thread was alive until it exited:


{% highlight text %}
root@ubuntu-phablet:/tmp# ./fnotifystat -f
Time     Event  PID  Info  Duration Process
18:23:46 clone 32350 thread          /usr/lib/firefox/firefox
18:23:46 clone 20258 parent          /usr/lib/firefox/firefox
18:23:46 clone 32351 thread          /usr/lib/firefox/firefox
18:23:46 clone 20258 parent          /usr/lib/firefox/firefox
18:23:46 clone 32352 thread          /usr/lib/firefox/firefox
18:23:46 exit  32350      0    0.079 /usr/lib/firefox/firefox
18:23:46 clone 20258 parent          /usr/lib/firefox/firefox
18:23:46 clone 32353 thread          /usr/lib/firefox/firefox
18:23:55 fork  32339 parent          bash
18:23:55 fork  32354 child           bash
18:23:55 exec  32354                 bash
{% endhighlight %}


Because of an ambiguity in the interface to the kernel, earlier versions of `forkstat` erroneously displayed every call to `clone` as a call to `fork` instead. Version 0.01.11 fixed this.

The most interesting command line options are:

* `-e` can be used to trace only a given list of events.

* `-S` shows cumulative event statistics when the command terminates.






## smemstat

Back when I explained the `RSS` and `SZ` fields shown by `ps`, we realised that it's not that easy to find out how much memory a process actually consumes in the end. Every process has its own address space, and in that address space there are several different regions: machine code, stack, private data, shared data, libraries, memory mapped files and devices, etc.

Stack and private data regions are easy, they can be accounted to the process in full because they differ between processes. But what about code, shared libraries and shared data? The kernel is clever, it realizes that it only has to load these into memory once if they are used by multiple processes at the same time, and then just has to pretend to every process that it has got its own copy. Memory mapped files and devices are not even real memory, just make-shift regions emulated by the kernel.

So if you sum up the size of all memory regions in all process address spaces, the number is usually *much* larger than the real memory consumption. If only there was a tool that handled all those shared regions correctly. `smemstat` to the rescue! Instead of just displaying mostly useless values like `RSS`, it tries to paint a more realistic picture by dividing the output into `USS`, `PSS` and `RSS`.

`USS` is the unique set size of a process. It counts every bit of memory in the process space that only exists once for this process in the system. `smemstat` is quite good at calculating this, for example if there is only one instace of the `bash` process running, it counts the whole code segment towards this single process. If the same binary is running muĺtiple times, `smemstat` sees that the kernel keeps only a single copy of the code segment and every shared libraries in memory, so it doesn't count these regions against the unique set size.

`PSS` is the proportional set size of a process. `smemstat` starts with the unique set size of the process and then adds a proportional piece of every shared area. Let's say there are three instances of `bash` running, and the code segment of the binary is exactly 900 kilobytes in size. Also `bash` needs the shared library `libc.so.6`, which is let's say exactly 1000 kilobytes in size, but is also used by two other processes in the system. `smemstat` will then account (900 / 3) = 300 kilobytes of the code segment and (1000 / 5) = 200 kilobytes for the shared library towards every running instance of `bash`.

`RSS` is, as already described, the total size of all areas in a process' address space *which are currently loaded in*, which means they have been accessed at least once and are not swapped out.

Let's look at the difference between the following two outputs to get a feeling for how this all works. I made sure that nothing is swapped out to make things a bit easier, especially on the bq Aquaris E4.5 swapping is the norm though, so you will often have to combine `Swap` and `RSS` in your mind.

Single `bash` instance running:


{% highlight text %}
  PID       Swap       USS       PSS       RSS User       Command
  2527     0.0 B  1160.0 K  1180.0 K  1780.0 K phablet    -bash
{% endhighlight %}


Multiple `bash` instances running:


{% highlight text %}
  PID       Swap       USS       PSS       RSS User       Command
  2527     0.0 B   528.0 K   751.0 K  1780.0 K phablet    -bash
  3943     0.0 B   504.0 K   727.0 K  1756.0 K phablet    -bash
  3713     0.0 B   504.0 K   727.0 K  1756.0 K phablet    -bash
{% endhighlight %}


If just one `bash` instance is running, most of it counts towards `USS`, and `PSS` isn't much higher than `USS` because `bash` only loads a small number of libraries which are used by nearly every other process in the system as well. If you divide a ~100 kilobyte library among 100 processes (this actually happens e.g. with `ld-2.21.so`), there's not much to account to a single process.

When we run three `bash` instances it gets more interesting: Note that `RSS` is always about the same value, because a freshly started `bash` in this example has loaded and accessed areas worth ~1780 kilobytes. But now just about everything that is not unique to each `bash` process is accounted towards `RSS` and `PSS`, not against `USS`! What remains for `USS` are mostly heap and stack. `PSS` is much lower than before, because the code segment can be divided by three. Shared libraries are negligible in this example.

With those three values a developer can now actually see what's really going on. When optimizing for low memory consumption, `USS` should obviously be kept as low as possible for example. A high `RSS`+`Swap` might be okay if `PSS` is much lower, because in that case there might be a lot of shared regions which are shared with many other processes, lowering the "impact" of every single process.

You will obviously only get to see your own processes if you don't run `smemstat` as root. The most important command line switches are:

* `-k`, `-m` and `-g` to report memory in kilobytes, megabytes or gigabytes, respectively.

* `-o` to dump the data to a JSON formatted file.

* `-p` to monitor a list of processes.



If you know better and/or something has changed, please do find me on the Freenode IRC or on Launchpad.net and get in contact!

[canonical-colin]: http://kernel.ubuntu.com/~cking/

[colin-ppa]: https://launchpad.net/~colin-king/+archive/ubuntu/white

[colin-ppa-mirror]: http://ppa.launchpad.net/colin-king/white/ubuntu/
