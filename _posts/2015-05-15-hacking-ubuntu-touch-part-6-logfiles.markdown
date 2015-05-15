---
layout: post
title:  "Hacking Ubuntu Touch, Part 6: Logfiles"
date:   2015-05-15 20:30:00
categories: Ubuntu Touch
---

*NOTE: This is a continuation of the [series]({% post_url 2015-05-07-hacking-ubuntu-touch-index %}) and relies on having [Developer mode]({% post_url 2015-05-08-hacking-ubuntu-touch-part-4-developer-mode-and-ADB %}) enabled.*

Debugging usually begins with logfiles. An Ubuntu Touch device is a "normal" Ubuntu system at heart, and many processes write their logs to the usual places, but there are many differences.


## Log targets and locations

Let's look at where log messages are stored.


### The Android logging system

As we know by now, most Ubuntu Touch devices are repurposed Android devices and have to run Android kernels and a couple of Android bits (fenced off into an LXC container). The Android developers needed to provide a logging facility which is available from the very beginning of system startup, long before any storage is mounted, so they came up with their own, kernel-assisted logging sub-system. It consists of the following parts:

* The `logger` kernel driver sets up four character devices and associates each with an internal buffer.

* Processes directly write their messages to the character devices or use the `liblog` abstraction library.

* The `log` binary can also be used as an easy way to write log messages.

* The `logwrapper` binary executes a given command and re-routes `STDOUT` to the selected logging character device.

* The `logcat` binary reads the messages back from the kernel buffers, much like `dmesg`, but `logcat` also has built-in filtering.

* `adb` has a `logcat` command to query and filter the buffers. It uses `logcat` as a backend.


There are four available character devices and buffers:

* `main` is the target for application messages. The character device path is `/dev/log_main`, the size of the buffer is 64 kilobytes.

* `events` is the target for system events. The character device is `/dev/log_events`, the size of the buffer is 256 kilobytes.

* `radio` is the target for radio baseband messages. The character device is `/dev/log_radio`, the size of the buffer is 64 kilobytes.

* `system` is the target for low-level system debug messages. The character device is `/dev/log_system`, the size of the buffer is 64 kilobytes.


All buffers except the `events` buffer contain free-form ASCII text messages, `events` uses a binary format. The binary messages consist of a tag code followed by parameters, the tag codes are stored in `/system/etc/event-log-tags` (`/android/system/etc/event-log-tags` on Ubuntu Touch). This is the current content on my device after a normal boot (notice the first two lines):

{% highlight text %}
root@ubuntu-phablet:~# cat /android/system/etc/event-log-tags 
42 answer (to life the universe etc|3)
314 pi
2718 e
2720 sync (id|3),(event|1|5),(source|1|5),(account|1|5)
2740 location_controller
2741 force_gc (reason|3)
2742 tickle (authority|3)
2747 contacts_aggregation (aggregation time|2|3), (count|1|1)
3000 boot_progress_start (time|2|3)
3020 boot_progress_preload_start (time|2|3)
3030 boot_progress_preload_end (time|2|3)
20003 dvm_lock_sample (process|3),(main|1|5),(thread|3),(time|1|3),(file|3),(line|1|5),(ownerfile|3),(ownerline|1|5),(sample_percent|1|6)
50000 menu_item_selected (Menu type where 0 is options and 1 is context|1|5),(Menu item title|3)
50001 menu_opened (Menu type where 0 is options and 1 is context|1|5)
50021 wifi_state_changed (wifi_state|3)
50022 wifi_event_handled (wifi_event|1|5)
50023 wifi_supplicant_state_changed (supplicant_state|1|5)
52000 db_sample (db|3),(sql|3),(time|1|3),(blocking_package|3),(sample_percent|1|6)
52001 http_stats (useragent|3),(response|2|3),(processing|2|3),(tx|1|2),(rx|1|2)
60000 viewroot_draw (Draw time|1|3)
60001 viewroot_layout (Layout time|1|3)
60002 view_build_drawing_cache (View created drawing cache|1|5)
60003 view_use_drawing_cache (View drawn using bitmap cache|1|5)
60100 sf_frame_dur (window|3),(dur0|1),(dur1|1),(dur2|1),(dur3|1),(dur4|1),(dur5|1),(dur6|1)
65537 netlink_failure (uid|1)
70000 screen_toggled (screen_state|1|5)
70200 aggregation (aggregation time|2|3)
70201 aggregation_test (field1|1|2),(field2|1|2),(field3|1|2),(field4|1|2),(field5|1|2)
75000 sqlite_mem_alarm_current (current|1|2)
75001 sqlite_mem_alarm_max (max|1|2)
75002 sqlite_mem_alarm_alloc_attempt (attempts|1|4)
75003 sqlite_mem_released (Memory released|1|2)
75004 sqlite_db_corrupt (Database file corrupt|3)
78001 dispatchCommand_overflow
80100 bionic_event_memcpy_buffer_overflow (uid|1)
80105 bionic_event_strcat_buffer_overflow (uid|1)
80110 bionic_event_memmov_buffer_overflow (uid|1)
80115 bionic_event_strncat_buffer_overflow (uid|1)
80120 bionic_event_strncpy_buffer_overflow (uid|1)
80125 bionic_event_memset_buffer_overflow (uid|1)
80130 bionic_event_strcpy_buffer_overflow (uid|1)
80200 bionic_event_strcat_integer_overflow (uid|1)
80205 bionic_event_strncat_integer_overflow (uid|1)
80300 bionic_event_resolver_old_response (uid|1)
80305 bionic_event_resolver_wrong_server (uid|1)
80310 bionic_event_resolver_wrong_query (uid|1)
90100 exp_det_cert_pin_failure (certs|4)
{% endhighlight %}

All messages contain a severity field ("verbose", "debug", "information", "warning", "error"), the source's process ID and a tag (usually the name of the source, e.g. the process or app name).

Now let's look at some examples on how to use logcat to query and filter the log files. The tool's syntax and behavior is a bit "strange": You can specify one or multiple match rules in the format `tag:<priority>`. The priority is optional, and the tag may contain the special value `*` to match all tags. If you specify a priority, it will also match all higher priorities. An implicit "match all" rule will be added at the end of your rule list, so you have to use the special value `*:S` as the last rule in your command line to suppress the output of all lines not matched by the previous rules (!).

*(NOTE: The path to logcat is `/system/bin/logcat` on Android devices and `/android/system/bin/logcat` on Ubuntu Touch. You have to specify it in full on Ubuntu Touch because the path is not listed in the `PATH` environment variable.)*

Filter all buffers for messages of severity "warning" or higher:

{% highlight text %}
root@ubuntu-phablet:~# /android/system/bin/logcat *:W *:S
{% endhighlight %}


Filter all buffers for messages with the "AudioDigitalControl" tag and severity "debug" or higher:

{% highlight text %}
root@ubuntu-phablet:~# /android/system/bin/logcat AudioDigitalControl:D *:S
{% endhighlight %}


Filter the `radio` buffer for all error messages:

{% highlight text %}
root@ubuntu-phablet:~# /android/system/bin/logcat -b radio *:E *:S
{% endhighlight %}

*(NOTE: Reading from the `events` buffer [`/dev/log_events`] does not work on my Ubuntu Touch device, the action just blocks. This means that `logcat` also blocks.)*



### Kernel ringbuffer

The kernel ringbuffer can be read with the `dmesg` command as root, as usual. Depending on your device you might be a bit surprised by a number of "strange" messages like the following:

{% highlight text %}
[14535.247662] <MAGNETIC>  mag_context_obj ok------->hwm_obj->early_suspend=0 
[14539.721807] [Ker_PM][request_suspend_state]sleep (0->3) at 14539695042466 (2015-05-08 23:00:10.075304038 UTC)
[14539.721840] [Ker_PM][request_suspend_state]sys_sync_work_queue early_sys_sync_work
[14539.721884] [Ker_PM][request_suspend_state]suspend_work_queue early_suspend_work
[14539.746892] <MAGNETIC>  mag_context_obj ok------->hwm_obj->early_suspend=1
{% endhighlight %}

Ubuntu Touch kernels for phones and tables are most often based on Android kernels, and Android kernels are full of out-of-tree patches from Google and hardware vendors. That's where many of those strange messages are coming from.


### Android LXC container

All the Android bits run inside an LXC container, the logfiles for the LXC service can be found at `/var/log/upstart/cgmanager.log`, `/var/log/upstart/lxc.log`, `/var/log/lxc/android.log` and `/var/log/lxc/lxc-monitord.log`.


### syslog

Some system services write to the system log, which is stored at `/var/log/syslog`, or one of the more specialised logs like `/var/log/auth.log`. The syslog also aggregates the kernel ringbuffer so it becomes easier to correlate kernel messages with other activities.


### Upstart system session

Upstart is the current init system (soon to be replaced by systemd) and is the parent of most system services. When a service gets started, Upstart re-routes its `STDOUT` to a process-specific logfile in the `/var/log/upstart/` directory.


### Upstart user session

Much like Upstart manages system services, it also starts many user services like the `mediascanner-service` or the `media-hub-server`. The output of these processes gets re-routed to process-specific logfiles in the `/home/phablet/.cache/upstart/` directory.


### Unity8

The display server and dash will log to `/home/phablet/.cache/upstart/unity8.log` and  `/home/phablet/.cache/upstart/unity8.log`, respectively.

Unity8 re-routes the `STDOUT` of its sub-processes (e.g. the indicators) and of all running Click apps to app-specific log files in `/home/phablet/.cache/upstart/`. The filename of the Click app logfiles is `application-click-${NAME}_app_${VERSION}.log`. Note that there are no timestamps and the app just appends to the file if restarted. I use either `tail -f` or delete the file before every app start.


### System image update client

The log file is located at `/var/log/system-image/client.log`. There were several occasions where the updater refused to update the system image to the most recent release, in these cases this log can be crucial.




## Overview

This is a list of all running processes on my bq Aquaris E4.5 Ubuntu Edition (OTA-3, image r21) after the phone has booted normally (plus `adbd` / `sshd` for phablet-shell), and the location of their respective logfile(s).

*NOTE: This is as complete as possible, but do not take it as a reference.*

|---
| Source | Logfile |
|-|-|
| Kernel | `dmesg` and `/var/log/syslog`
| Authentication and authorisation | `/var/log/auth.log`
| AppArmor | `/var/log/upstart/apparmor.log.log`
| Apport crash logs | `/var/log/upstart/`
| `6620_launcher` | None?
| `account-polld` | `/home/phablet/.cache/upstart/account-polld.log`
| `accounts-daemon` | `/var/log/auth.log`
| `adbd` | `/var/log/upstart/android-tools-adbd.log`
| `address-book-service` | `/home/phablet/.cache/upstart/address-book-service.log`
| `app` | None?
| `bluetoothd` | `/var/log/upstart/bluetooth-touch.log`
| `camera_service` | Android Logging
| `ccci_fsd` | Android Logging
| `ccci_mdinit` | None?
| `cgmanager` | `/var/log/lxc/android.log`
| `cgproxy` | None?
| `ciborium` | `/home/phablet/.cache/upstart/ciborium.log`
| `cron` | `/var/log/syslog` and `/var/log/auth.log`
| `dbus-daemon` | `/home/phablet/.cache/upstart/dbus.log`
| `dconf-service` | None?
| `debuggerd` | None?
| `dm_agent_binder` | None?
| `dnsmasq` | None?
| `drmserver` | None?
| `drvbd` | None?
| `evolution-addressbook-factory` | None?
| `evolution-calendar-factory` | None?
| `evolution-source-registry` | None?
| `gsm0710muxd` | None?
| `healthd` | None?
| `history-daemon` | None?
| `indicator-bluetooth-service` | None?
| `indicator-datetime-service` | `/home/phablet/.cache/upstart/indicator-datetime.log`
| `indicator-display-service` | None?
| `indicator-location-service` | `/home/phablet/.cache/upstart/indicator-location.log`
| `indicator-messages-service` | `/home/phablet/.cache/upstart/indicator-messages.log`
| `indicator-network-service` | `/home/phablet/.cache/upstart/indicator-network.log`
| `indicator-power-service` | `/home/phablet/.cache/upstart/indicator-power.log`
| `indicator-secret-agent` | None?
| `indicator-sound-service` | `/home/phablet/.cache/upstart/indicator-sound.log`
| `indicator-transfer-service` | None?
| `init` | None?
| `installd` | Android Logging
| `lightdm` | `/var/log/lightdm/lightdm.log`
| `lxc-start` | `/var/log/lxc/android.log`
| `maliit-server` | `/home/phablet/.cache/upstart/maliit-server.log`
| `matv` | Android Logging
| `mdlogger` | Android Logging
| `media-hub-server` | `/home/phablet/.cache/upstart/media-hub.log`
| `mediascanner-service-2.0` | `/home/phablet/.cache/upstart/mediascanner-2.0.log`
| `memsicd3416x` | None?
| `mission-control-5` | None?
| `mnld` | Android Logging
| `mobile_log_d` | Android Logging
| `mtk_agpsd` | Android Logging
| `mtkbt` | None?
| `MtkCodecService` | Android Logging
| `mtp-server` | `/home/phablet/.cache/upstart/mtp-server.log`
| `netdiag` | Android Logging
| `NetworkManager` | `/var/log/upstart/network-manager.log`
| `nuntium` | `/home/phablet/.cache/upstart/nuntium.log`
| `nvram_agent_binder` | Android Logging
| `obexd` | `/var/log/syslog`
| `ofonod` | `/home/phablet/.cache/upstart/ofono-setup.log`
| `pay-service` | `/home/phablet/.cache/upstart/pay-service.log`
| `polkitd` | `/var/log/syslog` and `/var/log/auth.log`
| `posclientd` | None?
| `powerd` | `/var/log/upstart/powerd.log`
| `ppl_agent` | Android Logging
| `pulseaudio` | `/var/log/syslog`
| `rild` | Android Logging
| `rsyslogd` | `/var/log/upstart/rsyslog.log` and `/var/log/syslog`
| `rtkit-daemon` | `/var/log/syslog`
| `scoperegistry` | `/home/phablet/.cache/upstart/scope-registry.log`
| `sensorservice` | Android Logging
| `servicemanager` | Android Logging
| `slpgwd` | None?
| `smartscopesproxy` | None?
| `sshd` | `/var/log/auth.log`
| `sync-monitor` | `/home/phablet/.cache/upstart/sync-monitor.log`
| `systemd-logind` | `/var/log/dmesg.log` and `/var/log/auth.log`
| `systemd-udevd` | `/var/log/syslog`
| `telepathy-ofono` | None?
| `telephony-service-approver` | None?
| `telephony-service-handler` | None?
| `telephony-service-indicator` | `/home/phablet/.cache/upstart/telephony-service-indicator.log`
| `thermal` | Android Logging
| `thumbnailer-service` | None?
| `tonegend` | None?
| `trust-stored-skeleton` | None?
| `ubuntu-espoo-service` | `/var/log/upstart/ubuntu-espoo-service.log`
| `ubuntu-location-serviced` | `/var/log/upstart/ubuntu-location-service.log`
| `ubuntu-push-client` | `/home/phablet/.cache/upstart/ubuntu-push-client.log`
| `udisksd` | None?
| `ueventd` | None?
| `unity8` | `/home/phablet/.cache/upstart/unity8.log`
| `unity8-dash` | `/home/phablet/.cache/upstart/unity8-dash.log`
| `unity-system-compositor` | `/var/log/lightdm/unity-system-compositor.log`
| `upowerd` | None?
| `upstart-dbus-bridge` | None?
| `upstart-event-bridge` | None?
| `upstart-file-bridge` | None?
| `upstart-local-bridge` | None?
| `upstart-property-watcher` | None?
| `upstart-socket-bridge` | None?
| `upstart-udev-bridge` | None?
| `urfkilld` | `/var/log/upstart/urfkill.log`
| `url-dispatcher` | `/home/phablet/.cache/upstart/url-dispatcher.log`
| `usensord` | `/home/phablet/.cache/upstart/usensord.log`
| `usermetricsservice` | None?
| `whoopsie` | `/var/log/upstart/whoopsie.log`
| `wpa_supplicant` | `/var/log/syslog`
| `zeitgeist-daemon` | None?
| `zeitgeist-fts` | None?



If you know better and/or something has changed, please do find me on the Freenode IRC or on Launchpad.net and get in contact.
