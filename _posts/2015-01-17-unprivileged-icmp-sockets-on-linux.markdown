---
layout: post
title:  "Unprivileged ICMP sockets on Linux"
date:   2015-01-17 12:17:31
categories: jekyll update
---

While working on a cheap [Network Scanner app for Ubuntu Touch][ubuntu-touch-network-scanner] I ran into the following problem.

A Network Scanner needs to ping hosts. Pinging means sending ICMP ECHO and receiving ICMP ECHOREPLY packets, and traditionally one can only create the necessary raw ICMP socket if one has root privileges or somehow got hold of the CAP_NET_RAW capability. Unprivileged users can usually ping hosts using a setuid `/bin/ping` binary.

Calling `/bin/ping` from Qt/QML and parsing its output (see [the code][ubuntu-touch-network-scanner-qprocess-implementation]) does work on my Ubuntu 15.04 development desktop (Kernel 3.18.0-9-generic), but doesn't when the app runs in confined mode on the Ubuntu Touch phone (Nexus 4, Ubuntu Touch r14, Kernel 3.4.0-5-mako). Why? Because you can't execute binaries that haven't been shipped with your app, and you obviously can't ship setuid binaries with your app. Luckily I stumbled across a Linux [kernel patch][icmp-kernel-patch] from 2010 which allows unprivileged users to create an ICMP datagram socket that is restricted to sending and receiving ICMP ECHO packets.

The necessary code to send an ICMP ECHO packet to localhost is as follows ([see also][unprivileged-icmp]):

{% highlight C %}
#include <stdio.h>
#include <stdlib.h>

#include <malloc.h>
#include <string.h>

#include <unistd.h>
#include <errno.h>

#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <netinet/ip_icmp.h>
#include <arpa/inet.h>


int main(int argc, char** argv)
{
    struct sockaddr_in addr;
    struct icmphdr icmp_hdr;
    char packetdata[sizeof(icmp_hdr) + 5];

    // Create a datagram ICMP socket
    int sock = socket(AF_INET, SOCK_DGRAM, IPPROTO_ICMP);

    if(sock < 0)
    {
        printf("socket() errno: %i\n", errno);

	return EXIT_FAILURE;
    }

    // Initialize the destination address to localhost
    memset(&addr, 0, sizeof(addr));
    addr.sin_family = AF_INET;
    addr.sin_addr.s_addr = htonl(0x7F000001);

    // Initialize the ICMP header
    memset(&icmp_hdr, 0, sizeof(icmp_hdr));
    icmp_hdr.type = ICMP_ECHO;
    icmp_hdr.un.echo.id = 1234;
    icmp_hdr.un.echo.sequence = 1;

    // Initialize the packet data (header and payload)
    memcpy(packetdata, &icmp_hdr, sizeof(icmp_hdr));
    memcpy(packetdata + sizeof(icmp_hdr), "12345", 5);

    // Send the packet
    if(sendto(sock, packetdata, sizeof(packetdata), 0, (struct sockaddr*) &addr, sizeof(addr)) < 0)
    {
        printf("sendto() errno: %i\n", errno);

	return EXIT_SUCCESS;
    }
    
    printf("ICMP ECHO packet sent successfully\n");

    return EXIT_SUCCESS;
}
{% endhighlight %}

So everything is fine now, right? Well, no. It works, but now the situation is the other way around: It works on the phone, while the `socket()` call fails with *EACCES* on the desktop. It took me a while to find out why.

The original patch was modified to only allow access to unprivileged ICMP sockets to a range of group ids. The range can be configured via sysctl and differs across devices and distributions:

{% highlight bash %}
# Ubuntu Touch r14
net.ipv4.ping_group_range = 0   2147483647

# Ubuntu 15.04 Vivid Vervet
net.ipv4.ping_group_range = 1   0

# Fedora 21
net.ipv4.ping_group_range = 1   0
{% endhighlight %}

So the current setting on Ubuntu Touch is to allow access to every user, while the desktop distributions seem to disable the feature completely.

It would be nice if the feature was enabled on all distributions and devices, otherwise I have to build multiple code paths and let the application find out which one (unprivileged ICMP sockets or a call to `/bin/ping`) works in the given environment.

[ubuntu-touch-network-scanner]: https://github.com/Sturmflut/ubuntu-touch-network-scanner
[ubuntu-touch-network-scanner-qprocess-implementation]: https://github.com/Sturmflut/ubuntu-touch-network-scanner/commit/854ef64b468299005dd0754c000455c7706b0bda
[icmp-kernel-patch]: http://lwn.net/Articles/420800/
[unprivileged-icmp]: https://github.com/Sturmflut/unprivileged-icmp