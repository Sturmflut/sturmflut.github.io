---
layout: post
title:  "Recipe for an Ubuntu Touch WhatsApp webapp"
date:   2015-02-01 15:00:00
categories: Ubuntu Touch
---

One can turn WhatsApp Web into an Ubuntu Touch webapp using the following Exec line:

{% highlight bash %}
Exec=webapp-container --enable-back-forward --store-session-cookies --user-agent-string="Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.36" --webappUrlPatterns=https://web.whatsapp.com/* https://web.whatsapp.com/ %u
{% endhighlight %}

But I won't publish anything to the store because WhatsApp still goes after third-party apps and I don't want to be responsible for users being blocked.

If you have an Ubuntu Phone and need an Instant Messenger, please use [Telegram][webogram] instead.


[webogram]: https://appstore.bhdouglass.com/app/com.ubuntu.developer.corasaaa.webogram