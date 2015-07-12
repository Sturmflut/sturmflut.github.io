---
layout: post
title:  "Panda Madness"
date:   2015-07-12 14:00:00
categories: Ubuntu
---

Okay, it's time to end the [panda madness][panda-love-uappexplorer] and clear things up a bit.

I didn't create Panda Love. I'm not an artist like fellow Insider Ronnie Tucker, I usually hack at low-level things all day and occasionally go and create mediocre apps like [Flood It][flood-it-uappexplorer].

One day I helped Alan Pope with his [Ubuntu HTML5 app template][github-html5-app-template] and he showed me [CodeCanyon][codecanyon], a website that links programmers and creators with potential buyers. There is a special category for HTML5 games that can be licensed for about 10 to 20 US-$. Usually they have been created with a (currently Windows-only, but version 3 will have Linux support) game creator tool called [Construct2][construct2], which has a HTML5 exporter. When you license the game, you usually get all the assets, the Construct2 source file(s), and a license to publish the work for one platform. You are allowed to change the assets and modify the source code. Not open source, but really fair in my opinion.

When I came across Panda Love, I immediately saw its qualities. Perfectly adapted for a device with a touch screen, retro graphics that would scale to any display size without visible artifacts, great gameplay, great music, enough levels for a good start and infuriating enough to keep you interested. Suitable for all ages. Alan had already licensed and published [Don't crash][dontcrash-uappexplorer] and [Beer Rush][beerrush-uappexplorer], so I knew that CodeCanyon was trustworthy, that Ubuntu as a platform could run these games satisfactorily, and that Alan could help me if any problems came up. So I invested 14 US-$ and got to work.

I had to work around some issues that will be removed with OTA-5, but aside from that the process was as easy as expected: export the game to HTML5 in Construct2, copy the output into the prepared HTML5 template, test and ship.

Now, just uploading the game into the store would have been enough, but I aimed higher. I wanted this game to become an example for what we have achieved as a platform. Running a fast-paced game like Panda Love means you've got audio, video, touch input, browser engine etc. up and running, and that everything is fast enough. But it also means that you've suddenly become a target platform for everybody out there who is building HTML5 games, and that's an incredible amount of people with an incredible amount of existing code and assets. They just have to be informed that Ubuntu exists and comes into question. To achieve that, the game has to become popular enough to be a demonstrator object.

So I sat down and came up with a couple of cheap marketing ideas. My reach on social media is very limited, so I have to plan and time everything I write carefully. For example I tried to post in the late afternoon (Europe), because that's the time when both Europe and the USA are awake and my posts don't get missed so easily. And I tried to build up a bit of a "hype", by posting pictures of the panda with questions like "Hm. Who's this guy?" or "There he is again! I wonder where he's going?". That worked quite well up to some degree, some people even got a bit mad because they started to combine the panda with some of the other announcements I've been hinting at and came up with interesting conspiracy theories.

When the hype peaked, I made sure that the game works on all commercially available phones and both OTA-4 and OTA-5 and published it to the store. Then I switched my marketing strategy from handing out hints to full-on madness. I announced the game on every channel that has Ubuntu phone users on it and replied to every comment (good or bad) that anybody posted. This didn't require much work, but it proved pretty effective: Within a day Panda Love was listed as the "Game of the Week" in the app store scope. People who I've never heard of and online news websites I've never heard of started to write reviews about it. The game just hit 301 users and 18 positive reviews. It works so good that it tricked some people (even Canonical developers) into believing that it was a native application, not "just" a HTML5 export from a game creator tool.

Overall this experiment was more than successful. We now have a nice amount of solid games in the store that we can show off to anybody who claims that Ubuntu is "not there" yet as a platform. We have demonstrated that a whole category of applications, HTML5 games, can be made to work on Ubuntu without any modifications. Cross-plattform HTML5 developers can support Ubuntu without additional work. It becomes even better: Construct2 is free for personal use (which includes the right to ship the created work for free), and with [gDevelop][gdevelop] there is an open-source alternative. We just showed a whole group of so-called "non-developers" that they can build great apps and games for Ubuntu without having to delve into QML/Qt/C++. Excellent work, everybody!

Oh, and this just in: the Ubuntu phone for India will most likely start selling at the end of July.



[panda-love-uappexplorer]: https://uappexplorer.com/app/pandalove.sturmflut

[flood-it-uappexplorer]: https://uappexplorer.com/app/com.ubuntu.developer.sturmflut.floodit

[github-html5-app-template]: https://github.com/popey/ubuntu-html5-template

[codecanyon]: http://www.codecanyon.net

[construct2]: https://www.scirra.com/construct2

[dontcrash-uappexplorer]: https://uappexplorer.com/app/dontcrash.popey

[beerrush-uappexplorer]: https://uappexplorer.com/app/beer-rush.popey

[gdevelop]: http://compilgames.net/
