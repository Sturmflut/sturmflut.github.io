---
layout: post
title:  "Hacking Ubuntu Touch, Part 5: adb shell vs. phablet-shell"
date:   2015-05-08 20:30:00
categories: Ubuntu Touch
---

*NOTE: This is a continuation of the [series]({% post_url 2015-05-07-hacking-ubuntu-touch-index %}) and relies on having [Developer mode]({% post_url 2015-05-08-hacking-ubuntu-touch-part-4-developer-mode-and-ADB %}) enabled. Thanks to Oliver Grawert for his input about `phablet-shell`.*

Most development work requires running a shell on the device. There are two ways to get one on an Ubuntu Touch device: `adb shell` and `phablet-shell`. How do they differ and which one is "better"? And can the experience be improved even further?


## adb shell

This is the usual method, and it works by having the ADB Client connect to the Server, telling the Server to open a stream to the `shell` destination offered by the Daemon, and the Daemon spawning a shell process which is connected to the stream.

The advantage is that this works without additional tools, if you have a working ADB setup you have `adb shell`.

There are several disadvantages though:

* The terminal doesn't adapt to the actual size of its window, the size is always 80 x 25 characters until you change it to something sensible with `stty rows <rows> cols <cols>`. So by default you can't even see all of the output of the `top` command.

* Integration with termcap is broken. Try editing a file with `nano`, you can't e.g. press the `Enter` key after typing in the filename the file should be saved to.


## phablet-shell

This binary is part of the Ubuntu `phablet-tools` package and tries to overcome the disadvantages of `adb shell`. It does this by starting an SSH server on the device via ADB, setting up a local ADB port forwarding (usually on port 2222), copying over the most recent SSH key to the device (for passwordless login) and then calling `ssh` to connect to the forwarded port.

Using `phablet-shell` as a shell has several advantages over `adb shell`:

* The terminal will adapt to the surrounding terminal window. Start `top` on the phone and then change the size of the terminal window, the output will fill the available space.

* All the fancy termcap things will work, you can e.g. use `nano`.

* You can automatically sync your current `~/.bashrc` to the device with the `--copy` parameter.


But there are also some disadvantages:

* If the most recent SSH key is protected by a password, and you don't have a properly set up SSH agent, you will be asked about passwords a lot.

* Performance is a bit lower because you are now running SSH over TCP over ADB over USB instead of just ADB over USB. For example `adb push` transfers at over 8 megabytes/s to my device while `scp` transfers at around 6.5 megabytes/s over the forwarded SSH port. This shouldn't make much of a difference though, we just want to transfer some text around.

At the end `phablet-shell` is usually the better choice for most use cases.


## SSH tricks with phablet-shell

Developer mode only enables the ADB server while the display is unlocked, so if you open and close a lot of shells you either have to continuously unlock the device, which is inconvenient, or disable the screen lock, which consumes energy. Using `phablet-shell` doesn't avoid the problem at first sight, because the command tries to open an `adb shell` on the device to check for a running SSH server even if another `phablet-shell` is already running and the port forwarding is already in place.

But `phablet-shell` never stops the SSH server it started, and also doesn't remove the ADB port forwarding.

So you can unlock your device, start `phablet-shell` once to set up the SSH server and port forwarding, exit `phablet-shell` and from then on use SSH clients directly. This should always work, even if the device locks. You can even simplify things, put the following block in your `~/.ssh/config`

{% highlight bash %}
Host phablet
Hostname localhost
Port 2222
User phablet
{% endhighlight %}

and then just type

{% highlight bash %}
$ ssh phablet

$ scp /local/path phablet:/remote/path

$ rsync -a -v /tmp/sync/ phablet:/tmp/sync/

{% endhighlight %}

The URL for your graphical file managers is `sftp://phablet/`.




If you know better and/or something has changed, please find me on Launchpad.net and do get in contact!
