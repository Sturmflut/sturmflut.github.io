---
layout: post
title:  "Ubuntu Insiders Hangout 01.06.2015"
date:   2015-06-02 08:00:00
categories: Ubuntu BayTrail
---

*NOTE: This is a rough translation from a [german article][ikhaya-mx4] I wrote with fellow Insider Sujeevan Vijayakumaran.*

About two weeks ago the Meizu MX4 with Ubuntu was launched on the chinese market. During a Google Hangout Meeting for "Ubuntu Insiders", Canonical communicated the current plans for the european market: the MX4 will be available in europe starting in the third week of June. It will be sold at a price of 299.99 € via a "flash sale" model (like the bq Aquaris E4.5), but this time users have to find "invites" within an interactive game on the Meizu website. There will only be a couple hundred invites per day available.

After the launch of the bq Aquaris E4.5 Ubuntu Edition, bq told the press that they don't have any plans about future Ubuntu devices. Apparently the Aquaris E4.5 sale was such a success though that they contacted Canonical just a few weeks later, and during the Hangout the launch of a second device, the Aquaris E5 HD, was announced. (*NOTE: [We already knew this]({% post_url 2015-05-16-the-other-bq-ubuntu-phone %})*). The hardware of the Aquaris E5 HD is, with the exception of the 5" FullHD display and a much better 13 megapixel camera, identical to the Aquaris E4.5. The device will be sold regularly at a price of 199.99 €, so no more flash sales.

Back during the Ubuntu Online Summit Mark Shuttleworth announced a fully converged device for 2015. At that time the manufacturer of this fabled device was still a secret, now we know more: it will be a "brand-new" bq device which is not on sale yet, and will also be released with Android. Only the Ubuntu version will support Convergence. The launch is planned around October.

There weren't any news regarding the US market.


## Software News

OTA-4 is close to being released and will [contain a lot of changes and features][ota-4], the most important one being a bump of the base system to Ubuntu 15.04.

Canonical is working on a long, long list of features to be released with future OTAs. The plan is to have an update every month.

Besides general performance improvements and work on even better power management (the E4.5 with Ubuntu already beats Android so hard, it's not even funny), a lot of work is being invested into the scopes system. For example users reported that navigation between scopes is sometimes confusing and takes too much effort.

Scopes will generally become easier to use and will adapt better to the user, also they will implement much more functionality. An example for this is social media: not only will it be possible to post, share and tag content directly from a Scope, without the need to start a separate app, audio and video content will also be played inline. The addition of new online accounts gets streamlined and will require less clicks.

A special feature of Ubuntu Touch are Aggregator Scopes: they pull content from multiple other scopes and combine it into a single display. For example the "News" scopes gets its content from the BBC scope, the El Pais scope etc. Currently the association between Aggregator scopes and their source Scopes is static, and the order of the displayed entries is static as well. With "Scopes Tagging" a developer can assign keywords to a Scope, and an Aggregator Scope will instantly start to pull content from all Scopes that match its own keywords. A new Scope for a newspaper could be tagged with the "news" keyword and would therefore be used as a source for the "News" Scope.

Future releases might allow the user to instantly and dynamically create Aggregator Scopes themselves, but I currently do not know how this will be implemented.

Other improvements affect the playback of media files. Not only will playback controls be added to the indicators, more options will also be available outside the Music app: the same playlist could then be managed by a Scope and the Music app at the same time.

But also the core system will see some interesting improvements: the phone can now act as a WiFi hot-spot and share its mobile data connection to other devices. The Camera app will get more features like basic editing capabilities and filters. If the internal FM radio module present in the Aquaris E4.5 will be usable in the future is currently unknown.


## Frameworks instead of Services

While Google constantly increases the number of Google services Android relies on, Canonical is apparently doing the exact opposite. After the shutdown of the Ubuntu One File Synchronization service in June 2014 the company is focusing on powerful frameworks to allow potential developers to easily port their apps and services.

Canonical is currently not working on more native apps and helps content partners and carriers by creating needed Scopes for them. If you've been hoping for something big like Telegram, that was probably an one-time thing.


## New partners and more developers

We have known for a while that Canonical is working on services and Scopes for the chinese market together with mobile carrier China Mobile. Apparently several thousand people participated in developer education programs and developer contests all over the country. The number of chinese apps and scopes in the app store is still negligible, which is probably because there were no devices available, but now that the MX4 is being sold the situation should improve.

Canonical also said that they are working with two other mobile carriers as well, but couldn't reveal any details.


[ikhaya-mx4]: https://ikhaya.ubuntuusers.de/2015/06/01/meizu-mx4-startet-in-europa-bq-aquaris-e5-offiziell-bestaetigt/

[ota-4]: https://insights.ubuntu.com/2015/05/29/phone-updates-may/