---
layout: post
title:  "An update on Ubuntu on BayTrail"
date:   2015-05-19 18:00:00
categories: Ubuntu BayTrail
---

You might have wondered why there's no new version of my "Installing Ubuntu on BayTrail tablets" how-to coming. Well, as it turns out we can't get the kernel and hardware to an usable and stable state regardless of what we're trying :/

The biggest problem is the eMMC flash controller designed by Intel which controls the internal storage. Apparently it has a major design flaw and sometimes stops working if DMA is used while the CPU is in a deep sleep state. On my Thinkpad Tablet 8 this usually happens after at most 20 minutes. The second problem is with some hardware timer, which runs at different frequencies instead of a constant one. You'll notice that everything is fine for a while, then suddenly the terminal cursor starts blinking faster and faster, the keyboard is polled too fast (every keypress is read multiple times) until suddenly everything jumps back to the normal frequency again.

Now kernel versions up to 3.14 will often work if passed the right combination of `sdhci.debug_quirks` and `sdhci.debug_quirks2` kernel commandline parameter values. The problem is that those values seem to differ between devices, even within the same device series. Another user with a Thinkpad Tablet 10 has been working for a while with kernel 3.14 and a combination of values which at the same time don't seem to work on my tablet. The timer is broken on these kernels.

Since kernel 3.16 no combination of `sdhci.debug_quirks` and `sdhci.debug_quirks2` parameter values seems to work. We don't know why, probably some kernel behaviour has changed. It would be possible to bisect the faulty commit, but it takes hours to test every step and we can't waste days and weeks of our lives just because Intel builds crappy hardware and doesn't care for good Linux support.

The timer has been patched at some point on the way to kernel 4.0 and Intel finally came up with a "solution" for the eMMC controller bug about three weeks ago: they [merged a patch][android-x86-patch-sdhci-acpi] into their Android-x86 kernel (in the "android-4.0" branch) which just disables the CPU's deepest sleep state at the right moments. This increases power consumption, but hey, if it works, who cares, right? And yes, it actually works. No eMMC lockups even after hours, stable timer.

I was halfway through installing Ubuntu 15.04 when the system suddenly no longer reacted to keyboard input, stopped responding to network packets and froze without any log message. This also happened to the other user with the Thinkpad Tablet 10. We suspect it is a thermal issue since it was over 30 degrees on that day and my tablet became very warm under high load. Also it seems to go away if the tablet is kept cool (he literally put his device in the freezer), but currently his tablet has been running for over a day after he only chilled it before turning it one, so the issue might also just be related to some calibration at boot-time.

My free time is limited, so it will likely take quite a while until everything has been figured out.


[android-x86-patch-sdhci-acpi]: http://git.android-x86.org/?p=kernel/common.git;a=commit;h=df2f12ff0c7985e1164ac197623ccb32e761ccc7