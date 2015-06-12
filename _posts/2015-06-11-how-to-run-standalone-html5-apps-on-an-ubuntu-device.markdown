---
layout: post
title:  "How to run standalone HTML5 apps on an Ubuntu device"
date:   2015-06-11 22:00:00
categories: Ubuntu Touch
---

*NOTE: Thanks to Alan Pope and Olivier Tilloy from Canonical for their contributions.*


A couple of days ago Alan Pope published a nicely done, simple and infuriating game to the Ubuntu App store: [Don't Crash][dontcrash-appexplorer]. I played it for a couple of minutes, came dangerously close to throwing my bq Aquaris E4.5 against a perfectly good wall and didn't really think about it much longer. Until I noticed that this is not a QML or Qt or SDL application, it's a webapp. A webapp that runs when the phone is offline, works in full-screen mode, has access to the tilt sensors (it tells you to put your phone in landscape mode if it isn't) and has perfectly fluid animations. Now how is that even possible?

Welcome to the world of HTML5 on Ubuntu.


## The basics

You might already know that the HyperText Markup Language (HTML) is a special "markup" language used to structure the content of a web page rendered by a browser. A HTML document itself is static, which is why browser vendors have quickly added support for the JavaScript programming language so web sites can have some interactive elements. Over the years our browsers have gotten much more powerful, and with the fifth version of the HTML standard, HTML5, the whole technology has now become so flexible that you can do just about *anything* if you use the right combination of HTML elements and JavaScript code. Fullscreen display? Check. Animations? Check. Audio and video? Check. Whole games that run in a browser? Check.

Ubuntu devices actually support several application types, but only two of them are prominent:

* "Native" apps are built using QML and/or Qt or some other "real" programming language/environment. Native apps run offline (maybe accessing online services on demand), are extremely powerful and integrate perfectly with the rest of the system - but you can easily get the impression that building a native app is complex and "hard".

* Webapps are nothing more than a kind of "bookmark" to a website. They just call the `webapp-container` helper binary, which is actually a full web browser without most of the usual UI elements. It loads the specified URL in fullscreen mode and gives the impression of a "native" app - until your network connection gets wobbly and/or you realise that webapps are a bit slow and don't integrate as well with the rest of the system as native apps. Building webapps is so easy, the process [has even been automated][webapp-generator].

This separation is so pronounced that even [uAppExplorer][appexplorer] offers different categories for both, because everybody knows that you want a real app and not just a webapp, right?


## The easy way: leveraging webapp-container

Let's see how we can work around the "negative" aspects associated with webapps while still using `webapp-container`:

* "Only work when online": Actually a well-designed webapp could use Local Storage to cache its most important assets locally, then it would just need a network connection once and would continue working when the device is offline. Or you can bundle all the HTML pages, JavaScript code, CSS files and other assets with your app and have `webapp-container` use the local copy, so no network connection needed at all during runtime.

* "Don't integrate well with the rest of the system": HTML5 and JavaScript offer many APIs which can be used to access device features. [In fact the list is so long][html5-javascript-api] that it easily rivals the runtime environments of many "real" programming languages. The Ubuntu SDK developers also provide a number of [additional HTML5 elements][html5-sdk] which can be used to access device content, use online accounts etc.

* "Slow": You would never believe how fast a standalone, locally installed HTML5 application can run. Even the bq Aquaris E4.5 can run fairly complex HTML5 games using only a single core. Webapps mostly appear to be slow because today's websites force you to download several megabytes of content just to see pictures of a cat.

* "No fullscreen mode": There are actually several solutions available. One is to pass the `--fullscreen` argument to `webapp-container`, the other is to use the HTML5 Fullscreen API.

There are two easy ways to build a HTML5 webapp for Ubuntu devices: Create a new HTML5 project using the Ubuntu SDK, or use Alan's [github template][github-popey-html5-template] and open it with Qt Creator. Note that HTML5 projects created using the SDK wizard currently still use the `ubuntu-html5-app-launcher` container, which doesn't use the modern Oxide browser engine and is considerably slower.

Then just start editing the `index.html` file, or copy over your existing HTML5 project, and run the project on your desktop or device. You can use all the amenities offered by Qt Creator, including the option to bundle everything into a click package.


## More flexibility: Using a QML WebView wrapper

`webapp-container` already offers a lot of flexibility, but there are some things that apparently can't be done: force an orientation, translate/rotate/scale all content etc. This is especially "annoying" when you are porting an HTML5 appplication because you don't want to modify the existing code for a single new platform. Instead it would be nice if you tell the browser window to do the necessary things, right?

A solution for this is to write a wrapper in QML. In this case the `qmlscene` binary interprets a QML file which uses a `WebView` component to render the actual HTML5 content. The `WebView` component offers lots of useful options and properties, as can be seen [in the documentation][webview-sdk] and in this example:

{% highlight text %}
import QtQuick 2.0
import Ubuntu.Web 0.2
import Ubuntu.Components 1.0
import QtQuick.Window 2.1

Window {
    id: window
    //visibility: Window.FullScreen

    WebView {
        //rotation: 90

        anchors.fill: parent

        url: Qt.resolvedUrl("www/index.html")

        preferences.allowUniversalAccessFromFileUrls: true
        preferences.allowFileAccessFromFileUrls: true
        preferences.localStorageEnabled: true
        preferences.appCacheEnabled: true
    }
}
{% endhighlight %}

Using the QML `WebView` you get fine-grained control over everything, e.g. look at the commented out `rotation` option. You can artificially and transparently force landscape mode by enabling it, because the whole content is rotated using full OpenGL hardware acceleration.

The `preferences` properties can be used to enable different things, like access to the local filesystem (which we obviously need), Local Store, the app cache etc.

I've set up a [project template on github][github-sturmflut-html5-webview] but am looking for more contributions, especially because I can only test it on the Aquaris E4.5.





If you know better and/or something has changed, please do find me on the Freenode IRC or on Launchpad.net and get in contact!


[dontcrash-appexplorer]: https://uappexplorer.com/app/dontcrash.popey

[webapp-generator]: https://developer.ubuntu.com/webapp-generator/

[appexplorer]: https://uappexplorer.com/

[html5-sdk]: https://developer.ubuntu.com/api/html5/current/

[html5-javascript-api]: http://html5index.org/

[github-popey-html5-template]: https://github.com/popey/ubuntu-html5-template

[github-sturmflut-html5-webview]: https://github.com/Sturmflut/ubuntu-html5-webview-template

[webview-sdk]: https://developer.ubuntu.com/api/apps/qml/sdk-14.10/Ubuntu.Web.WebView/
