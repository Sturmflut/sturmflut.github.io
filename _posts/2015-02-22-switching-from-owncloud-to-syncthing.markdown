---
layout: post
title:  "Switching from ownCloud to Syncthing"
date:   2015-02-22 22:00:00
categories: Linux
---

***NOTE: This is a record of my personal experiences with ownCloud. Your mileage may vary.***

About six months ago I set up an [ownCloud][owncloud] instance on one of my servers. Besides the calendar feature, I used file synchronization between my home desktop, my laptop and my desktop at work.

The experience has been pretty poor in all regards for the following reasons:

- There is only one server, introducing a classic single point of failure. Since ownCloud 7 it is possible to configure Server-to-Server sharing, which makes things a bit better, but the whole architecture is still too complex for my taste.

- Synchronization is very slow (https://github.com/owncloud/client/issues/209) regardless of link speeds and CPU power, which meant that at the end I would boot up my laptop about 20 minutes before leaving the house every morning, just to make sure that all files were synchronized in time for the commute. Then I would put the laptop on my desk at work, connect it to the network and go for a coffee while it took about ten minutes to synchronize a hundred kilobytes in 18 files. Over gigabit links. On multi-core CPUs. Yes, THAT slow.

- ownCloud makes Backup quite hard. You have to manually copy a couple of paths and dump a database.

- There have been combinations of client and server versions which would lead to disappearing files.

- The upgrade process is fragile, and ownCloud doesn't give much support for older versions, so you are pretty much forced to jump to every new version, including a jump to a new major version.

- There would often be conflicts, even though I always only edited files on one machine at a time.




## Switching to Syncthing

Setting up [Syncthing][syncthing] on Ubuntu is quite easy and boils down to the following steps:

* Add the [PPA][syncthing-ppa] on every node and install the syncthing package

* Create the init script/Upstart configuration/systemd unit and start the service

* Configure your nodes to trust each other and share the same directory

That's it. No single point of failure, no additional HTTP server, no additional database, nothing. My main share is currently synchronized to nine nodes.

I use the following Upstart configuration under `/etc/init/syncthing.conf`:

{% highlight bash %}
description "Syncthing P2P sync service"

start on (local-filesystems and net-device-up IFACE!=lo)
stop on runlevel [!2345]

env STNORESTART=yes
env HOME=/home/USER/
setuid "USER"
setgid "USER"

exec /usr/bin/syncthing

respawn
{% endhighlight %}




## Security

Syncthing encrypts transfers, but if somebody gets physical access to one of your nodes while it is still running he may get access to the unencrypted files. [EncFS][encfs] (mostly) solves this problem.




## Backing up the files

There is only a single path to back up: the one containing the files. I created a cronjob which creates a compressed tar file of the encrypted share and keeps seven older versions. On every node, at different timestamps throughout the day. So now instead of backing up my data on a single ownCloud server, I have a crazy number of automatic backups on nine nodes distributed over the whole world.




## Backing up the ownCloud calendar with git and Syncthing

I still use the ownCloud installation for the calendar feature. To back up the calendar I came up with the following solution: the ownCloud server runs a cronjob which downloads the calendar via HTTP, to a directory that is part of my Syncthing share. This directory is a git repository. After downloading, the server commits the file. If it has changed since the last run, git will create a new revision.

{% highlight bash %}
#!/bin/bash

cd ~/Sync/backups/owncloud-calendar

OUTPUT=$(wget --auth-no-challenge --http-user=USER --http-password=PASSWORD "https://server/owncloud/index.php/apps/calendar/export.php?calid=2" -O calendar.ics 2>&1)

if [ $? -eq 0 ]; then
        git add calendar.ics &>/dev/null
        git commit calendar.ics -m "New version" &>/dev/null
else
        echo "${OUTPUT}"
fi
{% endhighlight %}

So now I do not only have a backup of my ownCloud calendar inside a text file instead of a database, it is also versioned.




[syncthing]: http://syncthing.net/
[syncthing-ppa]: https://launchpad.net/~ytvwld/+archive/ubuntu/syncthing

[owncloud]: https://owncloud.org/

[encfs]: https://vgough.github.io/encfs/